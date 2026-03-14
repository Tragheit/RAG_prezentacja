# Strategie wektoryzacji (embedding)

## 1. Standardowe wektory gęste (Dense Embeddings)

To klasyczne podejście, od którego wszyscy zaczynają (np. `text-embedding-3`, modele z rodziny `sentence-transformers` jak `all-MiniLM-L6-v2`).

* **Jak to działa:** Cały chunk tekstu (niezależnie od tego, czy ma 10, czy 500 tokenów) jest kompresowany przez sieć neuronową (najczęściej przez operację *mean pooling* na ostatniej warstwie ukrytej) do pojedynczego, gęstego wektora o stałej długości (np. 384, 768, 1536 wymiarów).
* **Zastosowanie:** Wyszukiwanie semantyczne (zrozumienie kontekstu, synonimów).
* **Wada dla inżyniera:** Informacja ulega silnej kompresji (tzw. *information bottleneck*). Model gubi precyzyjne słowa kluczowe, numery seryjne czy rzadkie akronimy.

## 2. Wyuczone wektory rzadkie (Learned Sparse Embeddings - np. SPLADE)

BM25, to algorytm statystyczny. Jednak w nowoczesnym ML stosuje się wektory rzadkie wyuczone przez sieci neuronowe.

* **Jak to działa:** Model (np. SPLADE) mapuje dokument na przestrzeń słownika (np. 30 000 tokenów BERT-a). Wektor w większości składa się z zer, ale aktywuje wagi dla słów, które występują w tekście, **oraz** dla słów, które w nim nie występują, ale są silnie powiązane kontekstowo (tzw. *implicit query expansion*).
* **Zalety:** Świetnie radzi sobie z wyszukiwaniem dokładnych fraz (lexical search), a jednocześnie "rozumie", że jeśli w tekście jest "auto", to warto podbić wagę dla słowa "samochód".

## 3. Wektoryzacja sterowana instrukcjami (Instruction-Tuned / Task-Aware Embeddings)

Modele takie jak **BGE (BAAI General Embedding)** czy **INSTRUCTOR** zostały wytrenowane tak, aby adaptować swoją przestrzeń wektorową w zależności od zadanego zadania.

* **Jak to działa:** Do tekstu dodaje się prefiks z instrukcją.
* Przy indeksowaniu dokumentu w bazie: *"Represent the Wikipedia document for retrieval: [TEKST]"*.
* Przy wektoryzacji zapytania użytkownika: *"Represent the question for retrieving supporting documents: [PYTANIE]"*.


* **Zalety:** Model dynamicznie przesuwa wektor w przestrzeni w zależności od tego, czy robimy asymetryczny RAG (krótkie pytanie -> długi dokument), czy np. clustering (dokument -> dokument).

## 4. Architektura Późnej Interakcji (Late Interaction - np. ColBERT)

To absolutny "game-changer" w zaawansowanym IR
* **Jak to działa:** Zamiast kompresować cały dokument do jednego wektora, ColBERT generuje osobny wektor dla **każdego tokenu** w dokumencie i zapisuje je wszystkie w bazie. Podczas wyszukiwania, zapytanie również jest dzielone na wektory tokenów. Podobieństwo oblicza się za pomocą operacji MaxSim (Maximum Similarity):

$$S(q, d) = \sum_{i \in q} \max_{j \in d} (E_{q_i} \cdot E_{d_j})$$


* **Zalety:** Drastyczny wzrost precyzji (SOTA dla wielu benchmarków).
* **Wady:** Wymaga znacznie więcej miejsca w bazie wektorowej (zapisujemy wektor per token, a nie per chunk) i jest bardziej obciążające obliczeniowo podczas wyszukiwania.

## 5. Dostrajanie do domeny za pomocą uczenia kontrastowego (Domain-specific Fine-tuning)

Gotowe modele od OpenAI czy Google są świetne do wiedzy ogólnej. Ale co, jeśli budujecie RAG dla dokumentacji medycznej albo specyficznego kodu korporacyjnego?

* **Jak to działa:** Inżynierowie ML biorą model open-source (np. z HuggingFace) i dotrenowują go na własnych danych. Używa się do tego **uczenia kontrastowego** (Contrastive Learning) z funkcją straty taką jak *Multiple Negatives Ranking Loss (MNRL)*. Model uczy się przyciągać do siebie wektory (zapytanie + trafny dokument) i odpychać (zapytanie + tzw. *hard negatives*, czyli dokumenty podobne, ale nieodpowiadające na pytanie).

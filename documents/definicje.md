## 1. Klasyczna reprezentacja tekstu i wyszukiwanie (Lexical Search)

### Bag of Words (Worek Słów)

To najprostsza metoda reprezentacji tekstu w NLP. Traktuje ona dokument jako "worek", do którego wrzucamy wszystkie zawarte w nim słowa, całkowicie ignorując ich kolejność, gramatykę czy kontekst. Liczy się tylko to, **czy** dane słowo wystąpiło i **ile razy**. Zbiór takich zliczeń tworzy wektor (zazwyczaj bardzo długi i wypełniony zerami – tzw. wektor rzadki).

![Bag of words](media/bow.png)

źródło: https://www.kaggle.com/code/abdallahwagih/nlp-bag-of-words-bow

* **Wada:** Zdania "Jan lubi psa" i "Pies lubi Jana" będą miały identyczną reprezentację, mimo odwrotnego znaczenia.

### TF-IDF (Term Frequency - Inverse Document Frequency)

To ulepszenie modelu Bag of Words. Zamiast tylko zliczać słowa, TF-IDF ocenia ich **ważność** dla konkretnego dokumentu w kontekście całej kolekcji (korpusu).
Składa się z dwóch części:

![TF IDF](media/tf_idf.png)

źródło: https://www.geeksforgeeks.org/machine-learning/understanding-tf-idf-term-frequency-inverse-document-frequency/

* **TF (Term Frequency):** Jak często słowo pojawia się w danym dokumencie.
* **IDF (Inverse Document Frequency):** Jak rzadkie jest to słowo w całym zbiorze dokumentów.
Dzięki temu słowa powszechne (np. "i", "oraz", "jest") otrzymują bardzo niską wagę, a słowa specyficzne (np. "chlorofil", "kwantowy") – wysoką.

### Algorytm BM25 (Best Matching 25)

To obecnie standardowy, klasyczny algorytm wyszukiwania używany w silnikach takich jak Elasticsearch czy Lucene.

![BM 25](media/bm25.png)

źródło: https://www.geeksforgeeks.org/nlp/what-is-bm25-best-matching-25-algorithm/

Jest to zaawansowana ewolucja TF-IDF, która wprowadza dwie kluczowe poprawki:

1. **Nasycenie częstotliwości (Term Frequency Saturation):** Jeśli słowo występuje 5 razy, jest ważne. Jeśli 50 razy, jest ważniejsze, ale nie 10 razy ważniejsze (krzywa rośnie coraz wolniej).
2. **Kompensacja długości dokumentu:** Dokument z 1000 słów siłą rzeczy będzie miał więcej powtórzeń danego słowa niż dokument ze 100 słowami. BM25 bierze pod uwagę średnią długość dokumentów w bazie, sprawiedliwie je oceniając.

---

## 2. Nowoczesna reprezentacja i wyszukiwanie (Semantic Search)

### Gęste wektory (Dense Embeddings)
W odróżnieniu od klasycznego podejścia (gdzie wektory mają wielkość całego słownika i w większości składają się z zer), nowoczesne modele (np. Word2Vec, BERT, OpenAI text-embedding) mapują tekst na **krótsze wektory o stałej długości** (np. 384, 768 lub 1536 wymiarów), gdzie każda liczba jest wartością ułamkową (gęstą).
Najważniejsze jest to, że te wektory **kodują znaczenie**. Słowa lub zdania o podobnym znaczeniu (np. "król" i "władca") znajdują się blisko siebie w tej wielowymiarowej przestrzeni matematycznej.

### Wyszukiwanie semantyczne (Semantic Search)
To podejście, które wykorzystuje gęste wektory do wyszukiwania informacji na podstawie **znaczenia i intencji**, a nie dokładnego dopasowania słów (keyword matching). Dzięki temu na zapytanie "Jak poprawić sylwetkę?" wyszukiwarka zwróci dokument o "Zasadach zdrowego odżywiania i treningu na siłowni", nawet jeśli żadne ze słów z zapytania nie występuje w artykule.

### Miara podobieństwa cosinusowego (Cosine Similarity)
Jak sprawdzić, czy dwa gęste wektory (np. wektor zapytania i wektor dokumentu) są do siebie podobne? Używa się do tego podobieństwa cosinusowego, które mierzy cosinus kąta między dwoma wektorami w przestrzeni wielowymiarowej.


$$\text{cosine\_similarity}(A, B) = \frac{A \cdot B}{\|A\| \|B\|}$$

* Wynik bliski **1** oznacza, że wektory (i teksty) są niemal identyczne (kąt to 0 stopni).
* Wynik bliski **0** oznacza brak związku (kąt prosty).

### Indeksowanie HNSW (Hierarchical Navigable Small World)
Gdy mamy miliony wektorów w wektorowej bazie danych, porównywanie zapytania z każdym z nich (krok po kroku) za pomocą podobieństwa cosinusowego byłoby zbyt wolne. HNSW to najpopularniejszy algorytm do **przybliżonego wyszukiwania najbliższych sąsiadów (ANN - Approximate Nearest Neighbor)**. Buduje on warstwowy graf – wyszukiwanie zaczyna się na najwyższej warstwie (gdzie jest mało punktów i długie połączenia), a następnie "schodzi" coraz niżej do lokalnych sąsiadów. Działa to błyskawicznie, niemal jak "szukanie po znajomych znajomych".

---

## 3. Łączenie i ocena wyników (RAG i Metryki)

### Re-ranker (np. Cross-enkodery)
Architektura wielu wyszukiwarek (np. w systemach RAG - Retrieval-Augmented Generation) jest dwuetapowa.

1. Najpierw szybki algorytm (BM25 lub HNSW) wyciąga ze 100 000 dokumentów np. najlepsze 100.
2. Następnie wchodzi **Re-ranker** (często oparty na modelu Cross-Encoder z rodziny BERT). Przetwarza on zapytanie oraz dany dokument **jednocześnie**, dzięki czemu rozumie głębokie zależności i kontekst między nimi. Obliczenia te są bardzo "ciężkie", dlatego odpala się je tylko dla tej finałowej setki, by precyzyjnie uszeregować je od najbardziej do najmniej trafnego.

### Metryki BLEU i ROUGE
Klasyczne metryki używane do oceny modeli generujących tekst (np. w tłumaczeniu lub streszczaniu), opierające się na dokładnym pokryciu słów i ciągów słów (n-gramów) pomiędzy tekstem wygenerowanym a referencyjnym.

* **BLEU (Bilingual Evaluation Understudy):** Skupia się na **precyzji**. Sprawdza, ile fragmentów z wygenerowanego tekstu (np. tłumaczenia) znajduje się w tekście wzorcowym. Typowo używane w tłumaczeniu maszynowym.
* **ROUGE (Recall-Oriented Understudy for Gisting Evaluation):** Skupia się na **czułości (recall)**. Sprawdza, ile ważnych fragmentów z tekstu wzorcowego modelowi udało się zawrzeć w wygenerowanym wyniku. Typowo używane do oceny streszczeń tekstów.

### BERTScore
To nowoczesna alternatywa dla BLEU i ROUGE. Zamiast wymagać dosłownego pokrycia słów, BERTScore wykorzystuje **gęste wektory (Embeddings z modelu BERT)**. Sprawdza podobieństwo cosinusowe pomiędzy każdym tokenem (słowem/fragmentem słowa) w wygenerowanym zdaniu, a tokenami w zdaniu referencyjnym.

* **Zaleta:** Jeśli tekst wzorcowy to "Pojazd jest szybki", a model wygeneruje "Auto jest prędkie", BLEU da wynik zerowy (brak pasujących słów), a BERTScore oceni ten wynik bardzo wysoko, bo zrozumie, że to synonimy o tym samym znaczeniu.
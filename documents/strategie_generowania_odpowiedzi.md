Oto zwięzłe, inżynierskie opisy tych zaawansowanych strategii. Są one idealne do sekcji "Advanced RAG / State of the Art" w Waszej prezentacji, ponieważ pokazują ewolucję od naiwnego wrzucania tekstu do bazy (tzw. *Naive RAG*) w stronę systemów produkcyjnych (*Advanced RAG*).

**1. Reranking (Dwuetapowe wyszukiwanie)**

Proces polegający na rozdzieleniu wyszukiwania na dwa etapy. Najpierw szybki, ale mniej precyzyjny retriever (np. baza wektorowa z HNSW) zwraca szeroką pulę dokumentów (np. Top 100). Następnie znacznie cięższy, ale bardzo precyzyjny model (Cross-Encoder, np. *Cohere Rerank*) analizuje każdą parę "zapytanie-dokument" jednocześnie, przypisując im dokładny wynik trafności i wybierając np. Top 5 dla LLM-a.

**2. Agentic RAG**

Podejście, w którym RAG nie jest sztywnym, jednokierunkowym potokiem (pipeline), ale jednym z **narzędzi (Tools)** dla autonomicznego Agenta LLM (często w architekturze *ReAct*). Agent sam analizuje problem, decyduje, czy musi odpytać bazę wektorową, tworzy odpowiednie zapytania, a jeśli wyniki są niewystarczające – modyfikuje zapytanie i szuka ponownie, dopóki nie skompletuje wiedzy potrzebnej do odpowiedzi.

**3. Knowledge Graph (Graph RAG)**

Zastąpienie (lub wzbogacenie) płaskiej bazy wektorowej grafową bazą danych. Dane są ekstrahowane jako relacje encji (np. `(Polska)-[jest_w]->(Europa)`). Znakomicie rozwiązuje to problem zapytań wymagających tzw. *multi-hop reasoning* (wnioskowania wieloskokowego), gdzie odpowiedź wymaga połączenia faktów rozsianych w różnych, niepodobnych do siebie semantycznie dokumentach.

**4. Contextual Retrieval**
Technika spopularyzowana m.in. przez firmę Anthropic. Przed wektoryzacją, oryginalny, mały chunk (np. "Przychody wzrosły o 20%") jest przepuszczany przez LLM, który analizuje cały dokument docelowy i dokleja do chunka kontekst (np. "Ten fragment dotyczy wyników finansowych firmy X w Q3 2023"). Taki "wzbogacony" chunk jest wektoryzowany, co drastycznie redukuje problem utraty znaczenia po podziale tekstu.

**5. Query Expansion (Rozszerzanie zapytania)**
Użycie LLM-a przed etapem wyszukiwania w celu przeformułowania pytania użytkownika. Pytania od ludzi są często krótkie i nieprecyzyjne. System automatycznie dopisuje do nich synonimy, rozwija skróty branżowe lub generuje hipotetyczne słowa kluczowe, aby zwiększyć szansę "trafienia" w odpowiednie wektory w bazie.

**6. Multi-query RAG**
Zamiast wysyłać jedno wektoryzowane zapytanie, LLM generuje np. 3 do 5 różnych wariantów tego samego pytania (pod innym kątem semantycznym). Wszystkie trafiają do bazy, a wyniki są agregowane (najczęściej przy użyciu algorytmu *Reciprocal Rank Fusion - RRF*). Minimalizuje to ryzyko, że model embeddingu nie zrozumiał specyficznej struktury oryginalnego zapytania.

**7. Context-aware chunking**
Dzielenie tekstu z poszanowaniem jego logicznej lub wizualnej struktury, a nie "na ślepo" co 500 znaków. Wymaga zaawansowanych parserów (np. *Unstructured.io*), które rozpoznają, gdzie kończy się sekcja HTML, gdzie zaczyna się definicja w Markdown, lub upewniają się, że tabela lub blok kodu JSON nie zostanie przecięty w połowie i zniszczony dla LLM-a.

**8. Late Chunking**

Bardzo nowatorskie podejście (spopularyzowane m.in. przez model *Jina Embeddings 3*). Tradycyjnie najpierw tniemy dokument, a potem wektoryzujemy chunki w izolacji. W *Late Chunking* cały dokument (np. 8000 tokenów) przechodzi przez model transformera (generując reprezentacje wektorowe *per token* z uwzględnieniem pełnego *self-attention* całego tekstu). Dopiero potem te "świadome kontekstu" wektory tokenów są uśredniane (mean-pooling) do postaci wektorów dla konkretnych chunków.

**9. Hierarchical RAG (Small-to-Big / Auto-merging)**
Strategia indeksowania na wielu poziomach. Dokument dzieli się na małe fragmenty (zdania/krótkie akapity), z którymi łączy się referencje do ich "rodziców" (całych stron lub sekcji). Wyszukiwanie wektorowe odbywa się na małych fragmentach (bo mają wysoką precyzję semantyczną dla konkretnego pytania), ale w fazie generacji LLM otrzymuje jako kontekst rozszerzony fragment nadrzędny, aby lepiej zrozumieć tło.

**10. Self-reflecting RAG (np. algorytmy CRAG / Self-RAG)**
Architektura posiadająca pętlę zwrotną. W toku działania wbudowane są mechanizmy krytycznej oceny (często w postaci dodatkowych, mniejszych klasyfikatorów). System weryfikuje na bieżąco: *Czy pobrane dokumenty są w ogóle na temat?* (jeśli nie, szuka ponownie). *Czy wygenerowana odpowiedź jest w pełni poparta faktami z kontekstu?* (jeśli nie, poprawia ją). Minimalizuje to ryzyko halucynacji.

**11. Fine-tuned Embedding (Dostrajanie modeli wektoryzujących)**
Gdy standardowe modele (OpenAI, standardowe berts z HuggingFace) nie rozumieją hermetycznego języka (np. nomenklatury prawnej, wewnętrznego kodu produkcyjnego firmy), inżynierowie ML dotrenowują wagi własnego modelu embeddingu metodą uczenia kontrastowego (Contrastive Learning). Wymaga to spreparowania dedykowanego zbioru par `(pytanie -> trafny dokument)`, ale pozwala wektorom idealnie odwzorować specyficzną dziedzinę.
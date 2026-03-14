Nowoczesna ewaluacja RAG dzieli się na dwie główne gałęzie: metryki niewymagające wzorca (tzw. Reference-Free, bazujące na LLM-as-a-Judge) oraz metryki oparte na wzorcu (Reference-Based).

## 1. Metryki paradygmatu LLM-as-a-Judge (Reference-Free)

*Te metryki nie wymagają "Ground Truth" (idealnej odpowiedzi od człowieka). Wystarczy zapytanie, odzyskany kontekst i wygenerowana odpowiedź. Sędzią jest zazwyczaj silniejszy model, np. GPT-4.*

* **Faithfulness (Wierność / Groundedness)**
* **Co mierzy:** Czy model uległ halucynacji? Sprawdza, czy wszystkie informacje w wygenerowanej odpowiedzi pochodzą *wyłącznie* z odzyskanego kontekstu.
* **Jak to działa:** Algorytm najpierw używa LLM-a do ekstrakcji wszystkich pojedynczych twierdzeń (tzw. *claims*) z odpowiedzi. Następnie sprawdza, ile z tych twierdzeń można logicznie wywieść z kontekstu.
* **Wzór:** 
$$Faithfulness = \frac{|Claims_{supported}|}{|Claims_{total}|}$$


* **Przykład z życia:** Jeśli kontekst mówi, że "Firma X zarobiła 1 mln", a odpowiedź mówi "Firma X zarobiła 1 mln w 2023 roku" (a roku nie było w kontekście) – Faithfulness spada, bo to halucynacja (nawet jeśli to prawda w świecie rzeczywistym!).


* **Answer Relevance (Trafność odpowiedzi)**
* **Co mierzy:** Czy odpowiedź faktycznie adresuje pytanie użytkownika, czy model "leje wodę" i odpowiada nie na temat?
* **Jak to działa:** Najpopularniejsza implementacja (np. w bibliotece Ragas) jest bardzo sprytna: bierze wygenerowaną odpowiedź i prosi LLM-a o wygenerowanie do niej 3 potencjalnych pytań (Reverse Question Generation). Następnie oblicza podobieństwo kosinusowe (Cosine Similarity) między tymi wygenerowanymi pytaniami a *oryginalnym* pytaniem użytkownika.
* **Zastosowanie:** Wykrywa sytuacje, gdy RAG odpowiada "Przepraszam, nie wiem" (co jest bardzo wierne kontekstowi, ale ma zerową trafność dla użytkownika).


* **Context Precision / Context Utilization (Precyzja Kontekstu)**
* **Co mierzy:** Sygnał do szumu (Signal-to-Noise Ratio) w promptach LLM-a. Z ilu dostarczonych chunków model faktycznie skorzystał?
* **Dlaczego to ważne dla ML:** LLM-y cierpią na zjawisko "Lost in the Middle" – jeśli wsadzimy im 10 chunków, a przydatny jest tylko jeden w środku, model może go zignorować. Ta metryka karze system za dostarczanie zbyt dużej ilości śmieciowego kontekstu.



---

## 2. Metryki oparte na Ground Truth (Reference-Based)

*Te metryki wymagają stworzenia zestawu testowego (Golden Dataset), w którym człowiek (Ekspert Domenowy) napisał idealną odpowiedź na dane pytanie.*

* **Answer Correctness (Semantyczna Poprawność)**
* **Co mierzy:** Jak bardzo wygenerowana odpowiedź pokrywa się faktograficznie z odpowiedzią wzorcową (Ground Truth).
* **Jak to działa:** LLM-Sędzia dzieli obie odpowiedzi na fakty i tworzy macierz pomyłek na poziomie semantycznym:
* **TP (True Positives):** Fakty obecne w obu odpowiedziach.
* **FP (False Positives):** Fakty wymyślone przez RAG (halucynacje).
* **FN (False Negatives):** Ważne fakty z odpowiedzi wzorcowej, które RAG pominął.


* Następnie obliczana jest miara F1 na tych faktach.


* **BERTScore**
* **Co mierzy:** Podobieństwo semantyczne między odpowiedzią wygenerowaną a wzorcową.
* **Jak to działa:** Zamiast porównywać słowo po słowie, BERTScore generuje wektory dla każdego tokenu w obu zdaniach (przy użyciu modeli z rodziny BERT) i oblicza ich podobieństwo kosinusowe.
* **Dlaczego jest świetny:** Zwykły algorytm uzna, że zdania "Auto jedzie szybko" i "Samochód porusza się prędko" to dwa różne zdania (0% pokrycia słów). BERTScore da im wynik bliski $0.95$, bo rozumie synonimy i ukrytą semantykę.



---

## 3. Klasyczne Metryki NLP (Anty-wzorce w RAG)

*Warto o nich wspomnieć jako przestrogę dla studentów. Bądźmy szczerzy – używanie ich do ewaluacji nowoczesnych LLM-ów to inżynieryjny błąd.*

* **ROUGE (Recall-Oriented Understudy for Gisting Evaluation) / BLEU (Bilingual Evaluation Understudy)**
* **Co to jest:** Stare metryki używane do tłumaczeń maszynowych i podsumowań. Mierzą dokładne pokrycie N-gramów (np. identycznych sekwencji 2 lub 3 słów) między wygenerowanym tekstem a wzorcem.
* **Dlaczego są złe dla RAG:** Są całkowicie "ślepe" na znaczenie. LLM z definicji jest potężnym silnikiem parafrazującym. Jeśli model udzieli perfekcyjnej, logicznej odpowiedzi, ale ułoży zdanie inaczej niż człowiek we wzorcu, ROUGE i BLEU dadzą mu tragicznie niski wynik.

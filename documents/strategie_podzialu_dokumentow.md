# Fixed Size Chunking (Podział o stałym rozmiarze)
Najprostsze podejście polegające na cięciu tekstu na sztywne bloki (np. po 500 tokenów) z określoną zakładką (tzw. overlap, np. 50 tokenów), aby zachować ciągłość między sąsiednimi fragmentami.

**Kiedy stosować:** 
Gdy zależy nam na maksymalnej szybkości indeksowania i mamy bardzo ograniczone zasoby obliczeniowe.

**Wady:** 
Ignoruje strukturę językową – często ucina zdania lub myśli w połowie, co drastycznie obniża jakość wektoryzacji i wyszukiwania.

# Recursive Chunking (Podział rekursywny)
Algorytm dzieli tekst hierarchicznie, używając uporządkowanej listy separatorów (np. ["\n\n", "\n", " ", ""]). Najpierw tnie po całych akapitach (\n\n). Jeśli akapit przekracza dopuszczalny limit tokenów, dzieli go dalej po pojedynczych zdaniach (\n), a w ostateczności po słowach ( ). Jest to standard np. w klasie RecursiveCharacterTextSplitter z LangChain.

**Kiedy stosować:** 
Jako domyślny, "złoty standard" dla większości projektów.

**Zalety:** 
Znacznie rzadziej rozbija sensowne wypowiedzi niż podział o stałym rozmiarze.

# Document Structure Based Chunking (Podział oparty na strukturze)
Podział wyznaczany przez naturalną, wizualną lub logiczną architekturę pliku. Wykorzystuje tagi HTML, nagłówki w Markdown (np. #, ##) lub układ stron i tabel w plikach PDF.

**Kiedy stosować:** 
Przy przetwarzaniu dokumentacji technicznej, artykułów naukowych, umów prawnych czy sformatowanych plików Markdown.

**Zalety:** 
Chunks mają doskonały sens semantyczny, bo odpowiadają konkretnym podrozdziałom. Wymaga to jednak dobrych parserów dokumentów (np. narzędzia Unstructured).

# Semantic Chunking (Podział semantyczny)
Zamiast polegać na znakach przestankowych, używa modeli uczenia maszynowego. Algorytm wektoryzuje (embedduje) pojedyncze zdania okno po oknie i mierzy ich podobieństwo (np. za pomocą cosine similarity). Jeśli podobieństwo między sąsiednimi zdaniami drastycznie spada, oznacza to, że autor zmienił temat – tam stawiana jest granica chunka.

**Kiedy stosować:** 
Gdy pracujemy z płynnym tekstem bez jasnej struktury nagłówków (np. transkrypcje wideo, długie eseje, rozmowy).

**Wady:** 
Jest znacznie wolniejsze i droższe obliczeniowo podczas fazy indeksowania, ponieważ wymaga ciągłego odpytywania modelu wektoryzującego.

# LLM Based Chunking (Podział oparty na LLM / Propositional Chunking)
Najbardziej zaawansowana metoda, wykorzystująca sam model językowy jako pre-procesor. LLM czyta dokument i systematycznie wyodrębnia z niego pojedyncze, niezależne fakty/stwierdzenia (tzw. propositions), które są następnie wektoryzowane niezależnie od oryginalnego tekstu. Czasami LLM używany jest też do generowania syntetycznych pytań/podsumowań dla każdego akapitu.

**Kiedy stosować:** 
W systemach wymagających najwyższej możliwej precyzji odpowiedzi, gdzie koszt infrastruktury schodzi na dalszy plan.

**Wady:** 
Skrajnie kosztowne czasowo i finansowo; trudne do skalowania na potężne zbiory danych (dziesiątki tysięcy dokumentów).
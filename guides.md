Statystyka medyczna RM2
Tomasz Przechlewski
Powiślańska Szkoła Wyższa (Kwidzyn/Poland)
t.plata-przechlewski@psw.kwidzyn.edu.pl
21/2/2026
Ankieta
Proszę o wypełnienie prostej i anonimowej ankiety:

https://docs.google.com/forms/d/e/1FAIpQLSd46zEcAdOwixhjV0vzik5xFhS2XtvaHT2n5FD037BAO1pD9A/viewform

dostępnej także przez stronę:

https://tprzechlewski.blogspot.com/2026/02/statystyka-medyczna-rm2.html

Sprawy organizacyjne
Doktorat zwalnia z udziału w zajęciach; ocena ze statystyki na studiach magisterskich może zostać przepisana

Zaliczenie przedmiotu: projekt wykonany w grupie maksimu 5 osób, oceniany przeze mnie oraz przez prowadzącego ćwiczenia.

Ważne: należy się podzielić na grupy oraz wybrać temat projektu przed 18 kwietnia 2026 (czyli przed następną sesją wykładów). Praca ma zawierać następujące elementy:

Określenie co jest analizowane + podanie źródła skąd dane zostały pobrane

Analiza jednej zmiennej np. częstość występowania otyłości w Europie

– histogram i/lub wykres pudełkowy lub jakiś inny,

– średnia, mediana, kwartyle,

– wariancja, odchylenie standardowe, odchyleni ćwiartkowe

– współczynnik skośności

Analiza dwu zmiennych, np. częstość występowania cukrzycy a występowanie otyłości w Europie albo number of neonatal deaths vs number of medical doctors.

– wykres (korelogram)

– model regresji liniowej, interpretacja parametrów(u) modelu

– współczynnik determinacji R2
, ocena istotności parametrów

Analiza szeregu czasowego, np liczby zgonów na szpiczaka plazocytowego w Polsce

– wykres

– model trendu liniowego

– współczynnik determinacji R2
, ocena istotności parametrów

– obliczyć średnie tempo zmian

Projekt ma się wzorować na analizie przedstawionej na wykładzie i ćwiczeniach. Nie zaleca się wykorzystania innych metod niż pokazane na zajęciach.

Temat i skład zespołu proszę wysłać na mój szkolny adres, tj. t.plata-przechlewski@ zaczynając temat od słowa RMMgr. W treści listu należy napisać jakie zmienne będą analizowne. Czy przykładowo ja bym swój projekt opisał jako:

Analiza częstości występowania otyłości w Europie na podstawie danych Obesity among adults, BMI >= 30, prevalence (crude estimate) (%) (https://www.who.int/data/gho/data/indicators/indicator-details/GHO/prevalence-of-obesity-among-adults-bmi--30-(crude-estimate)-(-))

Analiza zależności pomiędzy otyłością a cukrzycą; dane dotyczące cukrzycy także pochodzą z bazy gHO Diabetes, prevalence (%) (https://www.who.int/data/gho/data/indicators/indicator-details/GHO/prevalence-of-diabetes-age-standardized)

Analiza dynamiki zgonów na szpiczaka plazmocytowego. Dane z bazy WHO mortality (https://platform.who.int/mortality)

Czyli chodzi o to, żeby napisać jakie zmienne się będzie analizowało.

Materiały
Materiały na wykłady https://tprzechlewski.blogspot.com/

Dane
Podstawowa idea zajęć (wykład/ćwiczenia) jest taka, że dane są realnie a nie sztuczne (czyli uproszczone i oczyszczone) i pochodzą z nast baz danych:

WHO (różne wskaźniki dotyczące zdrobia) https://www.who.int/data/gho https://www.who.int/data/gho/data/indicators https://www.who.int/data/gho/data/indicators/indicators-index

WHO Mortality (zgony wg przyczyn) https://platform.who.int/mortality

FAO (Wyżywienie, środowisko) Na przykład FAO Food balance https://www.fao.org/faostat/en/#data/FBS

Bank Światowy

Eurostat (wiadomo)

BDL (np. zgony wg przyczyn ale szczątkowo moim zdaniem) BDL generalnie bez szału

Inne dane: wybór tematu projektu wykorzystującego inne dane, np. dane medyczne, które są dostępne w miejscu pracy wymaga skonsultowania tego z wykładowcą i uzyskania jego akceptacji do 18.04.2026.

Oprogramowanie
Arkusz kalkulacyjny Excel

program do obliczeń statystyczno-ekonometrycznych Gretl (w części dotyczącej analizy szeregów czasowych)

Plan wykładów
Opisowa analiza jednej zmiennej
Czego szukamy: rozkładu wartości, wartości typowej, zmienności (duża/mała)

Metody analizy:

tablica statystyczna (dla zmiennej jednowymiarowej numerycznej i nominalnej)

wykres (słupkowy, histogram, punktowy, box-plot)

parametry: średnia, mediana, dominanta (cecha nominalna) kwartyle, tercyle (zamiana na wartości nominalne mało-średnio-dużo) wariancja, odchylenie ćwiartkowe asymetria

Opisowa analiza dwóch zmiennych:
Czego szukamy: zależności jednej od drugiej

Metody analizy:

tablica dwudzielcza (krzyżowa)

wykres rozrzutu (skala liniowa i logarytmiczna)

parametry: współczynniki korelacji (Pearsona, Spearmana)

model regresji

Wprowadzenie do wnioskowania statystycznego
Czego szukamy: uogólnie wyników z próby na całą populację

Metody:

test dla wskaźnika struktury

testy dla średnich (t-, ANOVA, nieparametryczne)

tabela korelacyjna, test Chi-kwadrat

test dla współczynnika korelacji i regresji

Będziemy to testować na tych samych danych z WHO, np czy średnia czegoś z trzech różnych okresów jest istotnie różna; czy współczynnik korelacji jest istotnie różny od zera, czy współczynnik(i) nachylenia w modelu regresji itp…

Analiza szeregów czasowych
Czego szukamy: opisujemy zmianę poziom zjawiska (wartości zmiennej) w czasie

metody analizy:

tablica dla szeregu czasowego (trywialne)

wykres liniowy/słupkowy

średnie tempo zmiany (średnia geometryczna)

Prosty model trendu

Wygładzanie wykładnicze

Przykład analizy: Pobrać jedną zmienną wykonać analizę dynamiki poznanymi metodami, np pobrać liczbę zgonów na szpiczaka plazocytowego z bazy WHO Mortality.

Analiza przeżycia (tylko wykład)
Czego szukamy: analiza czasu, który upłynął do wystąpienia zdarzania

Metody analizy:

Krzywa przeżycia

Tabela życia (life table)

Model proporcjonalnych hazardów Coxa
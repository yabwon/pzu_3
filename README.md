# pzu_3
szkolenie z SASa, materiały



kod o datach:
~~~sas

/*DATA SASOWA: Liczba dni, które upłynęły od 1 stycznia 1960.*/

/*
itd. wstecznie
...
29 grudnia 1959 => -3
30 grudnia 1959 => -2
31 grudnia 1959 => -1
1 stycznia 1960 => 0
2 stycznia 1960 => 1
3 stycznia 1960 => 2
4 stycznia 1960 => 3
...
19 października 1993 => 12345
itd.
*/
/*
01jan1960
*/


data data1;
	d=1;
run;


data data2;
	d=12345;
	format d date9.;
run;

/*8 października 2025 ????*/

data date3;
	d = "8oct2025"d;
run;

data date4;
	d = "8oct2025"d;
	format d yymmdd10. ; /* date9. date11. */
run;


dAtA date5;
	/*d = "8oct2025"d;*/
	d = today();
	mu = "8Jan1935"d;
	format d mu yymmdd10. ; /* date9. date11. */

	LD = d - mu;
run;

data data6;
	a="ABC";
	b="abc";

	if A = B then put "równe";
	         else put "różne";
run;
~~~

datetime

~~~~sas
/* datetime a.k.a. timestamp */

/*DATA SASOWA: Liczba dni, które upłynęły od 1 stycznia 1960.*/

/*DATETIME SASOWY: Liczba sekund, które upłynęły od 1 stycznia 1960 od początku dnia. */

data data7;
	d = "8oct2025"d;
	dt = "8oct2025:12:34:56"dt;

	format d /*yymmdd10.*/ dt datetime.;
run;


data data8;
	d = 86400;
	dt = "8oct2025:12:34:56"dt;

	format d yymmdd10. dt datetime.;
run;


data data8;
	d = today();
	dt = "18oct2025:12:34:56"dt;

	dp = DATEPART(dt);

	format d dp yymmdd10. dt datetime.;
run;

data data8;
	d = today();
	dt = "18oct2025:12:34:56"dt;

	d2 = d*86400 + "12:34:56"t;
	d3 = DHMS(d, 12, 34, 56);

	format d yymmdd10. d2 d3 dt datetime.;
run;
~~~~

procesowanie DATA stepu

~~~~sas
data storm_complete;
	set pg2.storm_summary_small; 
	where Name is not missing;

	length Ocean $ 8;
	Basin=upcase(Basin);
	StormLength=EndDate-StartDate;

	if substr(Basin,2,1)="I" 
		then Ocean="Indian";
	else if substr(Basin,2,1)="A" 
		then Ocean="Atlantic";
	else Ocean="Pacific";

	drop EndDate;
run;

/* faza kompilacji */
/*
1) sprawdzenie składni
2) budujemy wektor PDV (Program Data Vector) * na podstawie wejściowych danych (SET,MERGE,itd.) oraz struktury kodu
3) ustalenie długości i kolejności zmiennych, 
4) ustalenie utrzymywania(KEEP) i wyrzucania(DROP) zmiennych
5) ustalenie czy zmienne będą reinicjalizowane brkiem danych (RETAIN)
6) filtrowanie danych na wejściu (WHERE)
7) budowa nagłówka (deskryptor, metadane) zbiru wynikwego (o ile jest tworzony)


>Name Char 15 - fitrowanie z WHERE  
>Basin Char 2   
>MaxWind Num 8   
>StartDate Num 8 DATE9. 
*EndDate Num 8 DATE9. - drop
>Ocean Char 8
>StormLength Num 8 - inicjalizacja brakiem danych

*_ERROR_ Num
*_N_ Num

*/

/* faza wykonania  */
/*
	wektor PDV
	pętla główna (implicite DATA step loop)
	output
*/


proc contents data=work.storm_complete;
run;
~~~~


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


/*

	if substr(Basin,2,1)="I" 
		then Ocean="Indian";
    else Ocean="blablabla";



Basin="EP"

"P"=?="I" -> fałsz 0

*/


data storm_complete;
	set pg2.storm_summary_small; 
	where Name is not missing;

	length Ocean $ 8;
	Basin=upcase(Basin);
	StormLength=EndDate-StartDate;

	select;
		when(substr(Basin,2,1)="I") Ocean="Indian";
		when(substr(Basin,2,1)="A") Ocean="Atlantic";
		otherwise                   Ocean="Pacific";
	end;

	drop EndDate;
run;

data storm_complete;
	set pg2.storm_summary_small; 
	where Name is not missing;

	length Ocean $ 8;
	Basin=upcase(Basin);
	StormLength=EndDate-StartDate;

	select;
		when(substr(Basin,2,1)="I") Ocean="Indian";
		when(substr(Basin,2,1)="A") Ocean="Atlantic";
		when(substr(Basin,2,1)="P") Ocean="Pacific";
		otherwise PUTLOG "nieznane dane";
	end;

	drop EndDate;
run;

data test;
	x=1;y=2;output;
	x=3;y=4;output;
run;

data test2;
	set test;
	z = x + y;
run;

/*
PUTLOG _ALL_;
*/

data test2;                             PUTLOG "1)" _ALL_;
	set test;                           PUTLOG "2)" _ALL_;
	z = x + y;                          PUTLOG "3)" _ALL_; PUTLOG;
run;




data test3;
	set test;
	z = x + y;
	OUTPUT;
run;


data test4;
	set test;
	z = x + y;
	OUTPUT;
	OUTPUT;
run;


data test5;
	set test;
	z = x + y;
	OUTPUT;
	z = x * y;
	OUTPUT;
run;



data test6_dodawanie test6_mnozenie;
	set test;
	z = x + y;
	OUTPUT test6_dodawanie;
	z = x * y;
	OUTPUT test6_mnozenie;
run;




data Indian Atlantic Pacific;
	set pg2.storm_summary_small; 
	where Name is not missing;

	Basin=upcase(Basin);
	StormLength=EndDate-StartDate;

	select;
		when(substr(Basin,2,1)="I") OUTPUT Indian;
		when(substr(Basin,2,1)="A") OUTPUT Atlantic;
		/*when(substr(Basin,2,1)="P") OUTPUT Pacific;*/
		otherwise; /*PUTLOG "nieznane dane";*/
	end;

	drop EndDate;
run;






data Indian2 Atlantic2 Pacific2;
	set pg2.storm_summary_small; 
	where Name is not missing;

	Basin=upcase(Basin);
	StormLength=EndDate-StartDate;

	select;
		when(substr(Basin,2,1)="I") OUTPUT Indian2;
		when(substr(Basin,2,1)="A") OUTPUT Atlantic2;
		when(substr(Basin,2,1)="P") OUTPUT Pacific2;
		otherwise PUTLOG "nieznane dane";
	end;

	/*drop EndDate;*/ 
	keep Basin StormLength;
run;



/*
drop=
keep=
*/



data Indian3 Atlantic3 Pacific3;
	set pg2.storm_summary_small; 
	where Name is not missing;

	Basin=upcase(Basin);
	StormLength=EndDate-StartDate;

	select;
		when(substr(Basin,2,1)="I") OUTPUT Indian3;
		when(substr(Basin,2,1)="A") OUTPUT Atlantic3;
		when(substr(Basin,2,1)="P") OUTPUT Pacific3;
		otherwise PUTLOG "nieznane dane";
	end;

	drop Basin EndDate; /* !!! */
	keep Basin StormLength; /* !!! */
run;




data 
Indian4 
Atlantic4(drop=Basin EndDate) 
Pacific4(keep=Basin StormLength)
;
	set pg2.storm_summary_small; 
	where Name is not missing;

	Basin=upcase(Basin);
	StormLength=EndDate-StartDate;

	select;
		when(substr(Basin,2,1)="I") OUTPUT Indian4;
		when(substr(Basin,2,1)="A") OUTPUT Atlantic4;
		when(substr(Basin,2,1)="P") OUTPUT Pacific4;
		otherwise PUTLOG "nieznane dane";
	end;
run;


data ALL_oceans;
	set pg2.storm_summary_small(drop=basin); 
	where Name is not missing;

	StormLength=EndDate-StartDate;
run;



data ALL_oceans;
	set pg2.storm_summary_small(drop=b:); 
	where Name is not missing;

	StormLength=EndDate-StartDate;
run;


/* SASHELP.class */

/* 
Utworz dwa zbiory w podzialen na dzieci powyzej 13 lat i ponizej,
w pierwszym zachowaj wage imie i plec
w drugim zachowaj wzrost imie i plec


*/
~~~~


praca domowa

~~~~sas
/* praca domowa */
/* zastanow sie (bez uruchamiania kodu) jaki bedzie wynik nastepujacego przetwarzania */

data test;
	x=1;y=2;output;
	x=3;y=4;output;
run;

data testPracaDomowa;
	output;
	set test;
	z = x + y;
	output;
run;
~~~~

~~~~sas
data have;
input x $1. y;
cards;
A 1
A 2
B 1
A 3
B 2
B 3
B .
C 1
C 2
A 4
C 3
C 4
;
run;


/* zsumujmy wszystkie wartosci zmiennej Y 
   i wypiszmy je na koncu procesowania
*/


data want; 
	set have;
	suma = suma + y; /* ? */
run;

/* jak sas pracuje z brakami danych ? */

data _null_;
x = . + 1;

y = . * -1;

z = . / 1;

t = 1 / .;

u = . ** 2;

w = . < 1;
putlog _all_;
run;



data want; 
	set have;
	RETAIN suma 0;
	suma = suma + y; /* ?? */
run;


data want; 
	set have;
	RETAIN suma 0;
	if y ^= . then
		do;
			suma = suma + y; 
		end;
run;

data want; 
	set have;
	RETAIN suma 0;
	suma = sum(suma, y); 
run;

data _null_;
x = sum(1,.,2);
y = 1 + . + 2;
putlog _ALL_;
run;

data _null_;
x = sum(.,.,.);
putlog _ALL_;
run;


/* SUM STATEMENT */
data want; 
	set have;
	suma + y; 
run;

/* zmienna_kumulujaca + wyrazenie; */

data want; 
	set have;
	suma + (y**2); 
run;

data want; 
	set have;
	suma + 1; 
run;

data want; 
	set have;
	suma + (y ^= .); 
run;
~~~~

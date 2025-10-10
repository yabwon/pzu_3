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

kumulowanie
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



data want; 
	set sashelp.class;
    h + height;
	w + weight;
	b + weight*weight / height;
run;

/*
bmi* -> (waga**2/wzrost)
*/



data want; /* _n_ */
	set sashelp.class end=_E_;
    h + (-height);
	w + weight;
	b + weight*weight / height;

	if _E_=1 then output;
	keep h w b;
run;

~~~~

BY-GROUP processing
~~~~sas
/*-----------------------*/

data have;
input x $1. y;
cards;
A 1
A 2
B 3
A 3
B 3
B 3
B .
C 1
C 2
A 4
C 3
C 4
D 16
;
run;

data want; 
	set have end=_E_;
	suma + y; 
	if 1=_E_ then output;
run;

/* BY-GROUP processing */
/* przetwarzanie w grupach */

/*

wejscie:
A 1
A 2
B 3
A 3
B 3
B 3
B .
C 1
C 2
A 4
C 3
C 4

sortowanie:
A 1
A 2
A 3
A 4

B 3
B 3
B 3
B .

C 1
C 2
C 3
C 4

suma w grupach

A 1 poczatek grupy
A 2
A 3
A 4 koniec grupy
---
  10

B 3
B 3
B 3
B .
---
  9

C 1
C 2
C 3
C 4
---
  10

wypisanie wyniku
A 10
B 9
C 10
D 16
*/

PROC SORT data=WORK.HAVE out=WORK.posortowany ;
BY x;
run; 

data want;
	set work.posortowany;
    BY x;
	putlog _ALL_;
run;

/* unikalne wartosci zmiennej x */
data UW_X;
	set work.posortowany;
    BY x;
	if first.x = 1 then output;
	keep x;
run;

/*
wybierz druga i trzecia najwieksza wartosc Y w kazdej grupie (o ile istaneja)
*/

PROC SORT data=WORK.HAVE out=WORK.posortowanyXY ;
BY x descending y ;
run; 

data max_2i3;
	set posortowanyXY;
	BY x descending y;
putlog _ALL_;
run;

data max_2i3;
	set posortowanyXY;
	BY x;
	
	if first.X = 1 then n=0;
	n+1;

	/*if N = 2 OR N = 3 then output;*/
	if N in (2 3) then output;
run;


/* mamy zbior HAVE */
/* skumuluj dane ze zbioru HAVE do posatci jak nizej:
A 10
B 9
C 10
D 16
*/
data kumulacja;
	set posortowanyXY;
	BY x;
	
	if first.X = 1 then suma=0;
	suma+y;

	if last.x;
	output;

	drop y;
run;


/* filtrowanie */

data filtr_zly; /* !!! */
	set posortowanyXY;
	BY x;
	
	where first.X = 1;
run;


data filtr_ok;
	set posortowanyXY;
	BY x;
	
	if first.X;
	output;
run;

/* grupy nie sortowane */


data haveNS;
input x $1. y;
cards;
A 1
A 2
A 3
A 4
C 1
C 2
C 3
C 4
B 3
B 3
B 3
B .
;
run;

data kumulacjaNS;
	set haveNS;
	BY x; /* !!! */
	
	if first.X = 1 then suma=0;
	suma+y;

	if last.x;
	output;

	drop y;
run;


data kumulacjaNS;
	set haveNS;
	BY x NOTSORTED; 
	
	if first.X = 1 then suma=0;
	suma+y;

	if last.x;
	output;

	drop y;
run;

~~~~

funkcje

~~~~sas
/* praca z funkcjami */

data have2;
  a1=411;
  a2=312;
  a3=213;
  a4=114;
  a5="Aa";
  a6="Bb";
  bx=100;
  by=200;
  bz=300;
  output;
run;

/*
data _null_;
x = funkcja(.....);

CALL wywolanie(.....); - rutyna(.....)
run;
*/

data want;
	set have2(keep=a:);
	put _all_;
	call sortn(of a1-numeric-a6);
	put _all_;
run;

/* pytanie od Karoliny, czy "numeric" jest konieczne? */
data want;
	set have2(keep=a:);
	put _all_;
	call sortn(of a1--a6); /* !!! */
	put _all_;
run;

/*
data want;
	set have2(keep=a:);
	
	call sortn(of a1-numeric-a6);
	
    array X[4] a1-numeric-a6;

	do i = 4 to 1 by -1;
		put X[i]=;
	end;
run;
*/

data want;
	set have2(keep=a:);
	s = sum(of a1-numeric-a6);
	m = mean(of a1-numeric-a6);
	n = n(of a1-numeric-a6);
	v = var(of a1-numeric-a6);
	put _all_;
run;

/* funkcje dla daty i czasu */

data _null_;
t = today();

put t=;

put t= yymmdd10.;

r=year(t);
m=month(t);
put r= m=;
run;

/*
INTNX(interval <multiple><.shift-index>, start-from, increment <, 'alignment'>)
*/

data _null_;
t = today();
put t=;
put t= yymmdd10.;

nt = INTNX("month",t,0,"end");
put nt= yymmdd10.;

nt = INTNX("week",t,0,"end");
put nt= yymmdd10.;

nt = INTNX("year",t,0,"end");
put nt= yymmdd10.;
run;


data want;
	set have2(keep=a:);
    L2 = largest(2, of a1-numeric-a6);
	put _all_;
run;

data _null_;
	call streaminit(12345);
	do i = 1 to 10;
		x = RAND('INTEGER',1,6);
		put x=;
	end;
run;

data _null_;
	x = 333 + 1/3;
	put x=;
	y = ROUND(x,0.01);
	put y=;

	y = ROUND(x,0.0001);
	put y=;

	y = ROUND(x,1);
	put y=;

	y = ROUND(x,100);
	put y=;

	i = INT(x);
	f = floor(x);
	c = ceil(x);

	put i= f= c=;
run;

/* datepart() i timepart() */

/* INTCK */

data _null_;
	d1 = today();
	d2 = today()-65;
	put d1= yymmdd10. d2= yymmdd10.;

	m = intck("month",d1,d2);

	put m=;
run;

data _null_;
	d1 = today(); /* 2025-10-09 */
	d2 = today()-15;
	put d1= yymmdd10. d2= yymmdd10.;

	md = intck("month",d2,d1,"D");
	put md=;

	mc = intck("month",d2,d1,"C");
	put mc=;
run;

data _null_;
	Y = (d1-d2)/365.25;
run;


/* funkcje na tekstach */
data _null_;
t = "Ala ma kota Filemona, i żółwia";

u = upcase(t);

l = length(t);
k = klength(t);

s = substr(t,5,10);

sc = scan(t, 4, ", ");

sc2 = scan(t, 5, ", ");
sc3 = scan(t, 5, ", ", "M"); /* a,b,c,,,f */

put (_ALL_) (=/);
run;

data _null_;
t = "Ala   ma   kota   Filemona,   i   żółwia";

c1 = compress(t," ","P");
c2 = compbl(t);
c3 = translate(t,"!!","Aa");
c4 = tranwrd(t,"Ala","Basia");

razem = CATX("$", of c1-c4); /* CAT CATS CATX CATQ */

put (_ALL_) (=/);
run;


/* PUT() i INPUT() */

data _null_;
	x = "234" + 1000;

	y = 123 !! "ABC";

	put x= y=;
run;


data data1;
	l = input("234", best.);

	x = l + 1000;

	put x=;

	t = put(123, 3.);

	y = t !! "ABC";

	put y=;
run;


data have;
	data='2025-10-09';
run;

data want;
	set have;

	d = intnx('month', data, 0, "end"); /* !!!!!!!! */

data want;
	set have;

	d = intnx('month', input(data, yymmdd10.), 0, "end");

	format d date11.;
run;

~~~~

formaty uzytkownika

~~~~sas
/*++++++++++++++++++++++++++++++++++++++++++++++++++*/

Proc FORMAT;
/* 4 typy
 f num
 f char
 i num
 i char
*/
/*
<value/invalue> <$> NAZWA (opcje)
spcyfikacja
w formie

klucz=wartosc
klucz=wartosc
...

<OTHER=>
;
*/
run;

Proc Format;
VALUE nowy

low-<0="ujemne"
     0="zero"
0<-high="dodatnie"
OTHER="brak danych"
;
run;

libname x "S:\workshop\test_formatu";
data x.test;
do i = -17, -1, -0.5, 0, 1, 42, ., .;
	put i= i= nowy.;
	output;
end;

format i nowy.;
run;

proc print data=x.test;
run;

options nofmterr;

/* format ze zbioru */

data slownik;
	klucz="C"; opis="Centrala"; output;
	klucz="P"; opis="Poznań"; output;
	klucz="K"; opis="Katowice"; output;
run;

data formatZeZbioru;
	set slownik end=_E_;

start = klucz;
label = opis;
FMTNAME="$zeZbioru";
typ="c";
output;

if _E_=1;
HLO="O";
label="INNE!!";
output;

run;

Proc Format cntlin=formatZeZbioru;
run;


data _null_;
	do i = "C","P","K","I";
		put i= i= zeZbioru.;
	end;
run;

/* dlaczego to jest fajne? */

data duzy;
id=1; faktura=123;output;
id=2; faktura=123;output;
id=3; faktura=123;output;
id=4; faktura=123;output;
id=3; faktura=123;output;
id=4; faktura=123;output;
id=1; faktura=123;output;
run;

data maly;
id=2;output;
id=4;output;
run;

data maly;
set maly end=_E_;
FMTNAME="maly";
typ="n";
start=id;
label=1;
output;

if _E_=1;
HLO="O";
label="0";
output;

run;




Proc Format cntlin=maly;
run;

data want;
	set duzy;
	where put(id,maly.) = "1";
run;
~~~~

petla (do-loop)

~~~~sas

data slownik;
	klucz="C"; opis="Centrala"; output;
	klucz="P"; opis="Poznan"; output;
	klucz="K"; opis="Katowice"; output;
run;

data test;
id = "C"; s='1jan2025'd; baza=123; z=1; output;
id = "P"; s='1feb2025'd; baza=117; z=3; output;
id = "K"; s='1apr2025'd; baza=34;  2; output;
run;

/* granica baza < 10 */

/*

DO i = <start> TO <koniec> BY <podbicie>;

	put i=;

END;

*/
data _null_;
	DO i = 15 TO 37 BY 5;

		put i=;

	END;

	put "po" i=;
run;


data _null_;
s = 15;
k = 37;
b = 5;

	DO i = s TO k BY b;

		put i=;

	END;

	put "po" i=;
run;

data have;
s = 15;
k = 37;
b = 5;
run;


data _null_;
set have;

	DO i = s TO k BY b;

		put i=;

	END;

	put "po" i=;
run;

data have;
s = 15;
k = 37;
b = 5;
output;

s = 1;
k = 3;
b = 1;
output;
run;


data _null_;
set have;

	DO i = s TO k BY b;

		put _N_= i=;

	END;

	put "po " _N_= i=;
run;

data test;
id = "C"; s='1jan2025'd; baza=123; z=1; output;
id = "P"; s='1feb2025'd; baza=117; z=3; output;
id = "K"; s='1apr2025'd; baza=34;  z=2; output;
run;

data want;
	set test;
	DO i = baza TO 10 BY -z;
		
		output;
		s+1;	
	END;

	format s yymmdd10.;
run;



data have2;
id ="ABCDEF";
  a1=11;
  a2=12;
  a3=13;
  a4=14;
  output;

id ="GHJIJK";
  a1=13;
  a2=15;
  a3=17;
  a4=19;
  output;
run;

/* od kazdej kwoty w kwartale chce odjac 5 */
data want;
	set have2;

	a1=a1-5;
	a2=a2-5;
	a3=a3-5;
	a4=a4-5;
run;

data want;
	set have2;
	
	ARRAY kwartal[4] a1-a4;


 	do i = 1 to 4;
		kwartal[i] = kwartal[i] - 5;	
	end;

output;
drop i;
run;



data have2_5;
id ="ABCDEF";
  a1=11;
  a2=12;
  a3=13;
  a4=14;
  a5=20;
  b1=100;
  output;

id ="GHJIJK";
  a1=13;
  a2=15;
  a3=17;
  a4=19;
  a5=123;
  b1=125;
  output;
run;

data want;
	set have2_5;
	
	ARRAY kwartal[*] a:; /* *) */


 	do i = 1 to dim(kwartal);
		kwartal[i] = kwartal[i] - 5;	
	end;

output;
drop i;
run;


data want;
	set have2_5;
	
	ARRAY kwartal[*] _numeric_; /* *) */


 	do i = 1 to dim(kwartal);
		kwartal[i] = kwartal[i] - 5;	
	end;

output;
drop i;
run;

/***************************************/
~~~~

while/until-loop

~~~~sas
/***************************************/


data test;
	id = 1; t="A B C D E"; output;
	id = 2; t="X Y Z"; output;
run;

/*

	do while ( <war.> );
		...
	end;

	do until( NOT<war.> );
		...
	end;


*/

data want;
	set test;

	i=0;
	do until( polisa = " " );
		i+1;
		polisa = SCAN(t, i, " ");

		if polisa = " " then output;
	end; /* ( polisa = " " ) */
drop t i;
run;

data want;
	set test;

	i=0;
	do until( 0 );
		i+1;
		polisa = SCAN(t, i, " ");

		if polisa = " " then leave;
			            else output;
	end;
drop t i;
run;
~~~~

~~~~sas
/*

do while(1);
	...
	<STOP>
end;

do i=1 by 0;
	...
	<STOP>
end;

do i=1 by 1;
	...
	i
	<STOP>
end;

*/

data want;
	set test;

	do i = 1 by 1 until( 0 ); /* ! uwaga to jest petla nieskonczona */
		polisa = SCAN(t, i, " ");

		if polisa = " " then leave;
			            else output;
	end;
drop t i;
run;
~~~~


~~~~sas
/* Łączenie tabel */
/* Laczenie tabel */

/* konkatenacja, sklejanie w pionie */

data A;
  id = 00; kod_lokalizacji = "CC"; output;
  id = 01; kod_lokalizacji = "OD"; output;
  id = 02; kod_lokalizacji = "MK"; output;
  id = 03; kod_lokalizacji = "ID"; output;
  id = 09; kod_lokalizacji = "ID"; output;
data B;
  id = 05; kod_lokalizacji = "CC"; output;
  id = 06; kod_lokalizacji = "OD"; output;
  id = 07; kod_lokalizacji = "MK"; output;
  id = 08; kod_lokalizacji = "MD"; output;
  id = 11; kod_lokalizacji = "ID"; output;
run;


data razem1;
	set A B;
run;

data C;
  ix = 12; kod_lokalizacji = "OD"; output;
  ix = 13; kod_lokalizacji = "MD"; output;
  ix = 14; kod_lokalizacji = "ID"; output;
run;

data razem2;
	set A B C;
run;

/*
id kod_lokalizacji ix _ERROR_ _N_
*/

/*
RENAME
RENAME=
*/
data razem2;
	set A B C( RENAME=( ix=id ) );
run;



data D;
  id = '15'; kod_lokalizacji = "CC"; output;
  id = '16'; kod_lokalizacji = "OD"; output;
run;



data razem3; /* !!!!! */
	set A B C( RENAME=( ix=id ) ) D;
run;

data razem3; 
	set A B C( RENAME=( ix=id ) ) D( RENAME=( id=id_c ) );
run;

data razem3; 
	set A B C( RENAME=( ix=id ) ) 
		D( RENAME=( id=id_c ) IN=jestem_w_D); /* IN= */

	/*putlog jestem_w_D=;*/

	if jestem_w_D=1 then id = input(id_c, best.) ; /* id <- ID_C jako liczbe */ 
	drop id_c; 
run;


data E;
  id = 17; kod_lokalizacji = "XYZ"; output;
run;

data razem4; 
	set A B E;  
run;

/*
id{8} kod_lokalizacji{2} _ERROR_ _N_
*/	

data razem4;
	length id 8 kod_lokalizacji $ 3;
	set A B E;  
run;
/*
id{8} kod_lokalizacji{3} _ERROR_ _N_
*/

data razem4B;
	set E A B ; /* ! */ 
run;
~~~~

proc append
~~~~sas
/* **** */


data baza;
  id = 00; kod_lokalizacji = "CC"; output;
  id = 01; kod_lokalizacji = "OD"; output;
  id = 02; kod_lokalizacji = "MK"; output;
  id = 03; kod_lokalizacji = "ID"; output;
  id = 09; kod_lokalizacji = "ID"; output;
data nowe;
  id = 05; kod_lokalizacji = "CC"; output;
run;

data baza;
	set baza nowe;
run;

/* proc append */
proc APPEND base=BAZA data=NOWE;
run;

proc APPEND base=BAZA data=NOWE( where= (id=05) );
run;

proc APPEND base=BAZA data=NOWE( where= (1=1) );
run;

~~~~

przeplot
~~~~sas
/* interleaving - przeplot */

data A;
  id = 00; kod_lokalizacji = "CC"; output;
  id = 01; kod_lokalizacji = "OD"; output;
  id = 02; kod_lokalizacji = "MK"; output;
  id = 03; kod_lokalizacji = "ID"; output;
  id = 09; kod_lokalizacji = "ID"; output;
  id = 11; kod_lokalizacji = "ID"; output;
data B;
  id = 05; kod_lokalizacji = "CC"; output;
  id = 06; kod_lokalizacji = "OD"; output;
  id = 07; kod_lokalizacji = "MK"; output;
  id = 08; kod_lokalizacji = "MD"; output;
  id = 11; kod_lokalizacji = "ID"; output;
run;


data razem5;
	set a(in=w_a) b(in=w_b);
	BY id;     /* <------------------------- !*/
	put _all_;
run;


data razem5B;
	set a(in=w_a) b(in=w_b);
	BY id; 
	
	if first.ID then ile_zbiorow=0;
	ile_zbiorow + (w_a+w_b);

	if last.id AND ile_zbiorow>1 then output;
	keep id ile_zbiorow; 
run;
~~~~

bonus
~~~~sas
data 
	test_202501
	test_202502
	test_202412
	test_202411
	test_202410
	;
	set sashelp.class;
run;

data ALL;
	set test_202410-test_202412;
run;

options noDSNFERR;
data ALL2;
	set test_202410-test_202502;
run;
options DSNFERR;

proc OPTIONS;
run;
~~~~

Merge

~~~~sas
/* merdżowanie zbiorów */
/* polecenie MERGE     */

/* 1-do-1 */

data p;
	a="ABC";
	b=42;
	c=3;output;
	c=2;output;
	c=1;output;
run;

data d;
	retain c 0 d "XYZ" e 123; 
	do c=1 to 3;
		output;
	end;
run;

proc sort data=p;
	by C;
proc sort data=d;
	by C;
run;

data razem1;
	MERGE p d;
	BY c;
run;


data razem1;
	MERGE p(in=w_P) d(in=w_D);
	BY c;
	put _ALL_;
run;


/* "niedopasowane" wiersze */

data p;
	a="ABC";
	b=42;
	c=1;output;
	c=2;output;
	c=3;output;
run;

data d;
	retain c 0 d "XYZ" e 123; 
	do c=2 to 4;
		output;
	end;
run;

data razem2;
	MERGE p(in=w_P) d(in=w_D);
	BY c;
	put _ALL_;
run;

data 
	razem_outer
	razem_inner
	razem_left
	razem_right
	;
	MERGE p(in=w_p) d(in=w_d);
	BY c;
	put _all_;
	
	output razem_outer;
	if w_p=1 AND w_d=1 then output razem_inner;
	if w_p=1           then output razem_left;
	if           w_d=1 then output razem_right;

run;


/* 1-do-N - jeden do wielu*/

data p;
	a="ABC";
	b=17;c=1;output;
	b=33;c=2;output;
	b=42;c=2;output;
run;

data d;
	retain c 0 d "XYZ" e 123; 
	c=1;d="XY";output;
	c=2;d="YZ";output;
run;

data razem3;
	merge p d;
	by c;
run;


/* UWAGA!!! */
/* wiele do wileu "nie działa jak w SQLu!!" */
/* merge "wiele do wielu" ma zupełnie inną mechanikę działania */

data p;
	a="ABC";
	b=17;c=1;output;
	b=33;c=2;output;
	b=42;c=2;output;
	b=99;c=2;output;
run;

data d;
	retain c 0 e 123; 
	c=1;d="XY";output;
	c=2;d="YZ";output;
	c=2;d="XX";output;
run;

proc sql;
	select *
	from 
	p join d
	on p.c=d.c
	;
quit;

data razem3;
	merge p d;
	by c;
run;

/* "różne takie sytuacje" */

data p;
	a="ABC";
	b=17;c=1;output;
	b=33;c=2;output;
	b=42;c=2;output;
run;

data d; 
	cc=1;d="XY";output;
	cc=2;d="YZ";output;
run;

data razem3;
	merge p d(rename=(cc=c));
	by c;
run;


data p;
	a="ABC";
	b=17;c=1;output;
	b=33;c=2;output;
	b=42;c=2;output;
run;

data d; 
	c=1;a="XY";output;
	c=2;a="YZ";output;
run;

data razem4;
	merge p d;
	by c;
run;



data p;
	a="ABC";
	b=17;c=1;output;
	b=33;c=2;output;
	b=42;c=2;output;
run;

data d; 
	c="1";d="XY";output;
	c="2";d="YZ";output;
run;

data razem5; /* ! */
	merge p d;
	by c;
run;

data d1; 
set d(rename=(c=cc));
c = input(cc,best.);
drop cc;
run;

data razem5;
	merge p d1;
	by c;
run;

/*----------------------------*/
/* trzy zbiory w jednym łączeniu (ta sama zmienna) */
data p;
	a="ABC";
	b=17;c=1;output;
	b=33;c=2;output;
	b=42;c=3;output;
run;

data d; 
	c=1;d="XY";output;
	c=2;d="YZ";output;
run;

data t; 
	c=2;e=12345;output;
	c=3;e=67890;output;
run;

data razem6;
	merge p d t;
	by c;
run;


/* trzy zbiory w jednym łączeniu, różne zmienne */
data p;
	a="ABC";
	b=17;c=1;output;
	b=33;c=2;output;
	b=42;c=3;output;
run;

data d; 
	c=1;d="XY";output;
	c=2;d="YZ";output;
run;

data t; 
	d="XY";e=12345;output;
	d="YZ";e=67890;output;
run;

data razem7;
	merge p d;
	by c;
run;

proc sort data=razem7;
by d;
run;

data razem8;
	merge razem7 t;
	by d;
run;
~~~~

zadanie
~~~~sas
/*
1)zbudujemy 3 zbiory
A) id (num, 1 100), nazwa "ABC" + ID
B) id (num, 1 200), oferta X Y Z T <- losowo
C) 4 obserwacje oferta X Y Z T  opis = "Oferta eXtremalna";
"Youpee!! oferta" "Zjefajna ofera" "Tragiczna+"

2) łaczenie wszystkiego!
*/


data A;
do id = 1 to 100;
	length nazwa $ 8;
	nazwa=CATX(" ","ABC",put(id,3.));
	output;
end;
run;

proc Contents data=A;
run;

data B;
call streaminit(1234567);
do id = 1 to 200;
	length oferta $ 1;
	i = RAND('integer',1,4);
	oferta = SCAN("X Y Z T",i," ");
	output;
end;
drop i;
run;

data C;
oferta="X"; opis="Oferta eXtremalna"; output;
oferta="Y"; opis="Youpee!! oferta"; output; 
oferta="Z"; opis="Zjefajna ofera"; output; 
oferta="T"; opis="Tragiczna+"; output;
run;
~~~~

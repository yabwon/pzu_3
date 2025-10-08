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

```lex
%{
	#include "y.tab.h"
	#include <ctype.h>
	#include <stdlib.h>
	extern double yylval;
%}


%% 
[ \t]+ { printf("Skip whitespaces\n"); }
[0-9]+(\.[0-9]*)? { 
		yylval = atof(yytext); 
		printf("Found number: NUM = %lf\n", yylval); 
		return NUM; 
	}
\n { printf ("Reached end of line\n"); return yytext[0]; }
. { printf ("Found other data \"%s\"\n", yytext); return yytext[0]; }
%%

int yywrap() {return 1;} 

```

```yacc
%{
#include <stdio.h>
#include <math.h>
void yyerror(char*s) {printf("%s\n",s);}
extern int yylex();
extern int yywrap();

%}

%define api.value.type {double}
%token NUM

%%
input: /**/ | input line
;
line:   '\n' | rpn '\n' {printf("%f\n",$1);}
      | error '\n' {yyerrok;}
;
rpn:    NUM			{ 
						$$ = $1;
						printf ("Recognized a number.\n");
					}
      | rpn rpn '+' {
						$$=$1+$2;
						printf ("Recognized '+' expression.\n");
					}
      | rpn rpn '-' {
						$$=$1-$2;
						printf ("Recognized '-' expression.\n");
					}
      | rpn rpn '*' {	
						$$=$1*$2;
						printf ("Recognized '*' expression.\n");
					}
      | rpn rpn '/' {
						if ($2 == 0) yyerror ("divide by zero");
						else $$ = $1 / $2;
						printf("Recognized '/' expression\n");
					}
	  | rpn rpn '^' { 
						$$ = pow ($1, $2);
						printf ("Recognized '^' expression.\n"); 
	  				}
	  | rpn 'n' { 
						$$ = -$1;
						printf ("Recognized Unary minus expression.\n"); 
	  				}
;
%%

int main() {
	printf("Enter the expression: ");
	yyparse(); 
	return 0;	
}
```

![[Pasted image 20250405151431.png]]
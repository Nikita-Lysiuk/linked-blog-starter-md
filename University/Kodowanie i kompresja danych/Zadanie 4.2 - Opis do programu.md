# ASCII 
`4A 7A 79 6B 3A 20 70 6F 6C 73 6B 69 20 28 29`
# OSI-8859-2 
`4a ea 7a 79 6b 3a 20 70 6f 6c 73 6b 69 20 28 bf f3 b3 e6 29`
# UTF-8
`4a c4 99 7a 79 6b 3a 20 70 6f 6c 73 6b 69 20 28 c5 bc c3 b3 c5 82 c4 87 29`

- Odpowiedzi na pytanie: 
	1.  Liczba bajtów pokrywa się z liczbą znaków występujących w 7-bitowym ASCII. Znaki polskie zostały całkowicie zignorowane, ponieważ w kodzie wskazałem, aby ignorować błędy podczas kodowania. 
	2. W pliku zakodowanym za pomocą UTF-8 liczba bajtów jest większa, ponieważ polskie znaki zajmują po 2 bajty, podczas gdy OSI-8859-2 koduje polskie znaki za pomocą jednego bajtu.
	3. 
		`C5BC - ż` - Część 1: `0xC5BC`
		`C3B3 - ó` - Część 1: `0xC3B3`
		`C582 - ł` - Część 2: `0xC582`
		Tak, całkowicie zgadzają się z moimi ręcznymi obliczeniami. 
## Zadanie 8

``` JS
const fast_pow = (x, n) => {
    if (n === 0) return 1;

    if (n % 2 === 0) {
        return fast_pow(x * x, n / 2);
    } else {
        return x * fast_pow(x, n - 1);
    }
}

console.log(fast_pow(2, 10));
```

``` Scheme
(define (fast_pow x n)
    (if (= n 0) 
        1
        
        (if (= (modulo n 2) 0)
            (fast_pow (* x x) (/ n 2))
            (* x (fast_pow x (- n 1))))))

(display (fast_pow 2 10))
(newline)

```

___
## Zadanie 9
```  python
def calc_pi(n):
    pi = 0
    for k in range(0, n):
        pi += 4 * ((-1)**k) / (2*k + 1)

    return pi

print(calc_pi(1000000))

  
# Użyłem python dla tego zadania, ponieważ on zajmuje mniej kodu dla takich zadań. Też w pythonie jest łatwiej zrobienie matematycznych operacji.

# Python jest językiem skryptowym, więc nie musze kompilować, pisać różnych mainów, tylko mogę policzyć to co chcę i wyświetlić wynik.

# Też mógłbym użyć javascript, ale wydaje mi się, że matematyczne operacje lepiej robić w pythonie.
```

## Zadanie 9
### Kod jest napisany w języku Rust.
``` Rust 
use std::time::Instant;
const SIZE: usize = 10_000_000;

fn sum_by_ref(arr: &Vec<i32>) -> i32 {
    arr.iter().sum()
}
 
fn sum_by_val(arr: Vec<i32>) -> i32 {
    arr.iter().sum()
}

fn main() {
    let arr: Vec<i32> = vec![1; SIZE];
  
    let start = Instant::now();
    let sum_ref = sum_by_ref(&arr);
    let duration_ref = start.elapsed();

    println!("By ref: {} ms, Sum: {}", duration_ref.as_millis(), sum_ref);


    let start = Instant::now();
    let cloned = arr.clone();
    let clone_duration = start.elapsed();

    let start = Instant::now();
    let sum_val = sum_by_val(cloned);
    let calc_duration = start.elapsed();
    println!("By val: Clone: {} ms, Calc: {} ms, Total: {} ms, Sum: {}",
             clone_duration.as_millis(), calc_duration.as_millis(),
             (clone_duration + calc_duration).as_millis(), sum_val);
} 
```
![[Pasted image 20250307172309.png]]


### Kod jest napisany w języku C++.
``` C++
#include <algorithm>  
#include <chrono>  
#include <iostream>  
#include <numeric>  
#include <vector>  
  
int sumWithRef(std::vector<int>& nums)  
{  
    return std::accumulate(nums.begin(), nums.end(), 0);  
}  
  
int sumWithValue(std::vector<int> nums)  
{  
    return std::accumulate(nums.begin(), nums.end(), 0);  
}  
  
int main()  
{  
    std::vector<int> nums(10000000, 1);  
  
    auto start = std::chrono::high_resolution_clock::now();  
    int sum = sumWithRef(nums);  
    auto end = std::chrono::high_resolution_clock::now();  
    std::cout 
    << "sumWithRef: " 
    << std::chrono::duration_cast<std::chrono::milliseconds>(end - start).count() 
    << "ms. Sum: " << sum << std::endl;  
  
    auto refTime = std::chrono::
			    duration_cast<std::chrono::milliseconds>(end - start).count();  
  
    start = std::chrono::high_resolution_clock::now();  
    sum = sumWithValue(nums);  
    end = std::chrono::high_resolution_clock::now();  
    std::cout 
    << "sumWithValue: " 
    << std::chrono::duration_cast<std::chrono::milliseconds>(end - start).count() 
    << "ms. Sum: " << sum << std::endl;  
  
	auto valueTime = std::chrono::
				duration_cast<std::chrono::milliseconds>(end - start).count();  
  
    std::cout 
    << "Difference: " 
    << std::abs(refTime - valueTime) 
    << "ms" << std::endl;  
}
```

![[Pasted image 20250307172639.png]]

Ogólnie, proces kopiowania danych wymaga czasu, co nie jest zaskakujące

## Zadanie 10

- **Statyczny zakres (leksykalny):**  
    Wypisze `3 2, 3 4, 4 6`.
    - W `g()` lokalne `x = 3`, globalne `y = 2` – stąd `3 2`.
    - `f()` w `g()` używa globalnych zmiennych: x rośnie z 2 do 3, y z 2 do 4 – więc `3 4`.
    - `f()` w `main()` dalej używa zmienej globalnej: x z 3 do 4, y z 4 do 6 – czyli `4 6`.  
        Wszystko zależy od definicji w kodzie, a nie od kolejności wywołań.
___
- **Dynamiczny zakres:**  
    Wypisze `3 2, 4 4, 3 6`.
    - W `g()` lokalne `x = 3`, globalne `y = 2` – mamy `3 2`.
    - `f()` w `g()` widzi lokalne `x = 3` z `g()`, dodaje 1 (daje 4), a globalne y rośnie z 2 do 4 – więc `4 4`.
    - `f()` w `main()` nie ma już `g()` w stosie, więc bierze globalne `x = 2`, dodaje 1 (daje 3), a y z 4 rośnie do 6 – wychodzi 3 6.  
        Tu zmienne zależą od tego, kto wywołał funkcję i co jest "na wierzchu" stosu.

Dla pewności sprawdziłem również kod w JS jako przykład języka ze statycznym scopingiem i Emacs Lint jako język z dynamicznym scopingiem.
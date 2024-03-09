In this section, let's see how we can use rust to design a web backend with high efficiency with the support of concurrency. First let's prepare some basic knowledge about concurrency in rust.Create a new 
project by using command "cargo new backend-gcd", then we add a dependency crat in the cargo.toml like following:
```r
[dependencies]
tokio={version="1.14.0", features=["full"]}
```
the tokio crat can help us write concurrent code easily, pay attention to the features=["full"], crat itself may has a big size, by default the crat will only export part of its all capacity, by doing this
we can reduce the compile time and the size of executable because we only integrate part of the crat into our onw code. But sometimes the default export of the crat may not meet our need. For example we 
need the time feature of the tokio crat but it is not exported by default, the features=["full"] tell the compiler to link all capability from tokio into our code, by doing this we can use its time feature
in the code, otherwise the compiler will report error for not having the time feature. Now let's see how we can use tokoi cocurrency support, in main.rs
```r
use std::{time};
use tokio;

async fn print_odd() {
    let total = 10;
    let mut count = 0;
    let mut odd = 1;
    let three_second = time::Duration::from_millis(3000);
    while count < total {
        println!("print odd value : {}", odd);
        odd += 2;
        count += 1;
        /*
        await suspend the function and give the excution power to 
        other async function 
         */
        tokio::time::sleep(three_second).await;
    }
} 

async fn print_even() {
    let total = 10;
    let mut count = 0;
    let mut even = 2;
    let three_second = time::Duration::from_millis(3000);
    while count < total {
        println!("print even value : {}", even);
        even += 2;
        count += 1;
        tokio::time::sleep(three_second).await;
    }
}

#[tokio::main]
async fn main() {
    let odd = print_odd();
    let even = print_even();
    println!("begin counting...");
    //the following println will run before the above two functions
    //println!("odd object: {}, even object: {}", odd, even);
    //begin the running of two async functions and wait untile both of them finish
    tokio::join!(odd, even);
    println!("here is the end...");
}

```

you can see the async keyword before those functions. async keyword means the given function can suspend its execution and give up the cpu run time for a given moment, the tokio::time::sleep(three_second).await will cause the execution to suspend and other async function can get the cpu time slice for running.

in main, when we call print_odd and print_even, these two functions will return an object called future, if you are familiar with js, you may think about the promise object, and the async and await are
quite the like with async and await for js. The line :" println!("begin counting...");" will get executed before any code in print_odd and print_even. The tokio::join!(odd, even) will cause the code in
these two functions to run and wait until their complete, and this function can only be called in an async function, but main can't have async keyword before it, that's why we need the #[tokio::main] 
annotation. Use cargo run to run the above code and we get the following result:
```r
begin counting...
print odd value : 1
print even value : 2
print even value : 4
print odd value : 3
print odd value : 5
print even value : 6
print even value : 8
print odd value : 7
print odd value : 9
print even value : 10
print even value : 12
print odd value : 11
print odd value : 13
print even value : 14
print even value : 16
print odd value : 15
print odd value : 17
print even value : 18
print even value : 20
print odd value : 19
```
you can see the excution of print_odd and print_even are interleaved with each other, and until both of them finish their execution can the main function finish itself.

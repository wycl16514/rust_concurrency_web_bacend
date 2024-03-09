In this section, we will try to make a web backend by using Rust. There are many backend framework provided by other languanges, for example express for Nodejs, Flex or Django and FastAPI in python. 
The counterpart for Rust is called actix-web, let's bring it to our project, fisrt we need to add it as dependency in cargo.Toml:
```r
[dependencies]
actix-web = "4.1"
serde = {version="1.0", features = ["derive"]}
```
serde is like an encoder or decoder, it is used for encoding data when we send data over the internet, and decode data when we receive binary data from the net, we will see its usage. Before buidling our
web backend server, we need to understand one important concept in Rust that is closure. Closure is just like function pointer in c, inline function in c++, lambda in python, and anonymous function in js,
we can assign a function like block to a veriable without executing the code, and we can pass this variable into a function, that means a function can receive a piece of code as its parameter and call 
that piece of code to complete some task, let's see an example:
```js
fn function_with_closure<G>(f: G)
where
    G: FnOnce(&str),
{
    f("hello world!");
}
fn main() {
    /*
    parameters passed in closure is using two bars ||
    */
    let s = "the content of x is :";
    let print_x_closure = |x: &str| {
        println!("{} {}", s, x);
    };

    function_with_closure(print_x_closure);

    function_with_closure(|x| {
        println!("{}: {}", s, x);
    });
}

```
from above code , print_x_closure is a closure, it defines a piece of code, it is different with normal function by its parameters, it uses two bars to receive param and normal function use brackets, and
closure can capture contex outside its body, for example the s variable is defined outsize of it, but when it is passed into function_with_closure as patameter, the s can still accessible in the body of 
function_with_closure, this is its amazing charm.

Let's see how we can defined a closure as parameter in funciton, first we need the FnOnece keyword, this tell the compiler we will pass a closure as parameter, and anything insize the brackets following
the FnOnce keyword is the type of parameter that can pass into the closure, the where is a keyword of Rust, it is a little be like where in sql, it defines the properties for a type, in the above code, 
it defines G is a type of closure with string literal as its parameter, and we can see we have two ways to pass closure as parameters, one is assign it to a variable and pass that variable to the function,
the other is we can define the closure in the place of the parameter when we calling that function. Running the above code you will get the following result:
```r
the content of x is : hello world!
the content of x is :: hello world!
```


Let's create a simple web server by using actix-web, and we will use closure in the code:
```r
use actix_web::{web, App, HttpResponse, HttpServer};

#[actix_web::main]
async fn main() {
    //initialize a web server instance and listening  requsts from frontend
    let server = HttpServer::new(|| App::new().route("/", web::get().to(get_index)));

    println!("server listening on http://localhost:3000...");
    server
        .bind("127.0.0.1:3000")
        .expect("error bidng server to port")
        .run()
        .await
        .expect("error running server");
}

async fn get_index() -> HttpResponse {
    HttpResponse::Ok()
        .content_type("text/html")
        .body("it works")
}
```

In the above code, we create a server instance and listening on port 3000 and binding it to localhost, you can see the .await behide the run, that means the server will suspend it self and release the cpu
to other and it will regain the cpu when time is right. What do we mean time is right? when there is a request coming from the web frontend, the server will wake up and will create an App istance in the
closure passed to HttpServer::new, the clousure in new will run for each request from the from end.

In the clousre, it will serve a http get request for the root and call the get_index to handle to request. get_index is an async function which means it can suspend itself. But in this case it just construct
an http ok response, and send back the string "it works" to the web browser, when you run the code above and open you browser, input "http://localhost:3000/" and hit return , you will get the following result:
<img width="343" alt="截屏2024-03-09 13 54 08" src="https://github.com/wycl16514/rust_concurrency_web_bacend/assets/7506958/2393b552-1921-4fb6-96fb-2417171a8716">


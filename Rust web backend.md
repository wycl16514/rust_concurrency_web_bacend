In this section, we will try to make a web backend by using Rust. There are many backend framework provided by other languanges, for example express for Nodejs, Flex or Django and FastAPI in python. 
The counterpart for Rust is called actix-web, let's bring it to our project, fisrt we need to add it as dependency in cargo.Toml:
```r
[dependencies]
actix-web = "4.1"
serde = {version="1.0", features = ["derive"]}
```
serde is like an encoder or decoder, it is used for encoding data when we send data over the internet, and decode data when we receive binary data from the net, we will see its usage. Before buidling our
web backend server, we need to understand one important concept in Rust that is closure. Closure is just like function pointer in c, inline function in c++, lambda in python, and anonymous function in js,
we can assign a function like block to a veriable without executing the code, and we can execute the code when time is right by using that variable, let's see an example here:
```js
fn print_x(x: i32) {
    println!("x value in function call is: {}", x);
}

fn main() {
    /*
    parameters passed in closure is using two bars ||
    */
    let print_x_closure = |x| {
        println!("x value in closure call is: {}", x);
    };

    print_x(1);
    print_x_closure(2);
}
```
from above code we can see the calling of closure and function are the same, for closure the passed in paramter are in two bars and for function are in brackets. Clousre will widly used as callback in application.


Let's create a simple web server by using actix-web:
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


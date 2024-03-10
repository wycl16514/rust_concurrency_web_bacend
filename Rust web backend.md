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

Now let's make the server more functional, we copy the gcd function to our project,and we change the get_index function to return more 
content like this:
```js
async fn get_index() -> HttpResponse {
    HttpResponse::Ok().content_type("text/html").body(
        r#"
        <title> GCD Calculator</title>
        <form action="/gcd" method="post">
        <input type="text" name="n"/>
        <input type="text" name="m"/>
        <button type="submit">Compute GCD</button>
        </form>
        "#,
    )
}
```
this time we return content in html format and browser can render it, one need to be notices is the
string fomat r#"", this kind of format is equivalence the reverse single quote `` in js, and the 
three single quote in python, in this string format, those special character such as ", / can be 
seem as normal character, and we don't need the slash to escape it.

let's see the result of this change:


![截屏2024-03-10 12 57 29](https://github.com/wycl16514/rust_concurrency_web_bacend/assets/7506958/5011900b-2d51-4e2b-8f6c-944930a2212d)

here we need user input two integers in two field controls, and hit the button, our server receive
the data and convert the two integers into a binary data structure that can be use by our code,
this time we need to use the serde crat, let's bring it into our project:
```r
using serde::Deserialize;
```
now we get the two integers sent by the frontend and convert it two a struct by deserialize the 
integers from string format into binary format like following:
```r
#[derive(Deserialize)]
struct GcdParameters {
    n: u64,
    m: u64,
}
```
we can see the attribute or directive above, this derictive will implicityly insert a piece of code
in the process of compiling, and those code can help us extract two integers from the http post 
form, actually the directive can generate code that help us extract data from many kind of format
like JSON, YAML, TOML.

Let's add code to handle the form data posted from the frontend:
```r
async fn post_gcd(form: web::Form<GcdParameters>) -> HttpResponse {
    if form.n == 0 || form.m == 0 {
        return HttpResponse::BadRequest()
            .content_type("text/html")
            .body("Computing the GCD with zero is boring");
    }

    let response = format!(
        "The greatest common divisor of the numbers {} and 
    {} is <b>{}</b>\n",
        form.n,
        form.m,
        gcd(form.n, form.m)
    );

    HttpResponse::Ok().content_type("text/html").body(response)
}
```
In above code, post_gcd will get the form data and check the value of m and n fields in the form,
if one of them is 0, we will return a http response with status code 400.We can see from the web::Form<GcdParameters>, here it will utilize code generate by the #[derive(Deserialize)],
and turn the form fields from frontend into binary struct GcdParameters.

If we get the two integers post by frontend , we call gcd to get their greatest common divisor and
then formating a html string and return it back the the browser. The foramt! macro can construct a
string by using place holder, finally we resgiter the post_gcd function to handle the post request
like following:
```r
 App::new()
            .route("/", web::get().to(get_index))
            .route("/gcd", web::post().to(post_gcd))
```
After the above code, when we input 48, 72 into the fields of the form and hit the button, we will
get the following result:

![截屏2024-03-10 13 38 17](https://github.com/wycl16514/rust_concurrency_web_bacend/assets/7506958/edf26dd8-dffa-41cd-88ff-8ebaf382c4f7)


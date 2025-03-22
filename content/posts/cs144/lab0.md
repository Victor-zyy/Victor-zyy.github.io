+++
title = "cs144-lab0"
author = ["zyy"]
date = 2025-03-21T15:58:00+08:00
tags = ["cs144", "c++"]
categories = ["net"]
draft = false
showtoc = true
+++

## 1 Set up Linux environment {#1-set-up-linux-environment}

1.  ubuntu 22.04
2.  g++ 11.4
3.  packages required

<!--listend-->

```sh
 $ sudo apt update && sudo apt install git cmake gdb build-essential clang \
clang-tidy clang-format gcc-doc pkg-config glibc-doc tcpdump tshark
```


## 2 Networking by hand {#2-networking-by-hand}

networking abstraction        ---reliable bidirectional byte stream


### 2.1 Fetch a Web Page {#2-dot-1-fetch-a-web-page}


#### steps {#steps}

-   [web server address](http://cs144.keithw.org/hello)
-   using telnet **telnet cs144.keithw.org http** to open a reliable byte stream between two computers.
-   First, Type **GET /hello HTTP/1.1** \`Enter\` the Enter is very important
-   Second, Type **Host: cs144.keithw.org** \`Enter\`
-   Third, Type **Connection: close** \`Enter\`\`
-   Finally, Type \`Enter\`


#### explanation {#explanation}

-------- **<http://host/path>**

-   GET tells the path part of the URL
-   Host tells the host part of the URL
-   close tells the server you finished the request
-   the last \`Enter\`tells the server you are done with the HTTP request

After that the result is as such.
![](/cs144/images/lab0_cs144_web_page_result.png)


### 2.2 Send yourself an email {#2-dot-2-send-yourself-an-email}

    send an email massage using a reliable byte stream to a service running on another computer
I use 163-netease email to do this job for I am not standford student.These steps are introduced as follows:

-   go to the setting of the 163-email to enable smtp server
-   Then using telnet to construct a connection between two computers
-   \`telnet smtp.163.com smtp\` or \`telnet smtp.163.com 25\` where port \`25\` is the smtp server port number
-   Type \`HELO hello.victor\` \`Enter\` it will echo \`250 OK\` the response code number starts with 2xx or 3xx is great to indicate connection is right! But when the response code number is 5xx, it tells you that something was wrong.
-   Type \`EHLO hello.victor\` it will print it out something you need to do to authentication.

{{< figure src="/cs144/images/lab0_EHLO_smtp.png" >}}

-   After that, you need to login in by type \`AUTH LOGIN\` , when you do that, you have to type ****base64**** coded you user name and vertification code.[base64 converter](https://www.base64decode.org/) This page will help you to do so, last but no least you have to set you own vertification code in your 163-email website.
-   When authentication is successfully, you are allowed to send an email.

{{< figure src="/cs144/images/lab0_result_smtp.png" width="300px" >}}

as you can see from the picture, after authentication, we can send email from byte stream by typing \`MAIL FROM:\` and \`RCPT TO:\`, don't forget to add an \`&lt;&gt;\`around your email address. When you type Data, you have to leave a blank line at the end of the headers. When finished the body, type \`Enter\`to end this edit. With all things done, type \`quit\` to end the conversation with the email.

telnet: a client that makes outgoing connection with programs running on other computers. Let's to be a simple server, which is kind of program that waits around for clients to connect it.


### 2.3 Listening and connecting {#2-dot-3-listening-and-connecting}

![](/cs144/images/lab0_netcat_server.png)
The netcat command \`v\` to produce more verbose output, and \`-l\` to listening a port and \`-p\` to designate port number 9090, the telnet to connect this server, which localhost indicates the same ip address shared each other.
![](/cs144/images/lab0_client_and_server.png)
This picture indicates this communication.


## 3 Writing a network program using an OS stream socket {#3-writing-a-network-program-using-an-os-stream-socket}

we are gonna to use the feature provided by operating system (Linux) that the ability to create a reliable bidirectional byte stream between two commputers.---- **Stream Socket**


### 3.1 webget {#3-dot-1-webget}

TCPSocket Class - in mindow project, the socket class lib is wrapped around using C++ and can be called from C code.


#### TCPSocket Class {#tcpsocket-class}

<!--list-separator-->

-  inheritance relationship

    this image shows up the inheritance relationship.

    {{< figure src="/cs144/images/lab0_webget_inheritance.png" width="300px" >}}

    <!--list-separator-->

    -  FileDescriptor Class

        -   copy/move constructor and assignment

        [Copy Constructor Ref Detail](https://en.cppreference.com/w/cpp/language/copy_constructor)

        A copy constructor is a constructor which can be called with **an argument of the same class type** and copies the content of the argument without mutating the argument.

        argument list must satisfy all following conditions

        1.  given the class type **T**, the argument list must be a reference of T
        2.  it will be T&amp;, const T &amp;, volatile T&amp;, const volatile T&amp;

        note: it can have multiple args but at least ref of class in it.

        When copy constructor is called?

        -   initialization T a = b; T a(b); where b is class of T.
        -   function arg passing void f(T a).
        -   function return value, which has no move constructor. T f(){ return a}

        [Copy Assignment Ref Detail](https://en.cppreference.com/w/cpp/language/copy_assignment)

        argument list must satisfy all following conditions

        1.  given the class type **T**, the argument list must be a reference of T and T
        2.  it will be T, T&amp;, const T &amp;, volatile T&amp;, const volatile T&amp;

        note: it must have one argument.

        When to call the copy assignment?

        The copy assignment operator is called whenever selected by overload resolution, e.g. when an object appears on the left side of an assignment expression.

        like: f1 = f2; operator= is selected when overload resoluting.

        [Move Constructor Ref Detail](https://en.cppreference.com/w/cpp/language/copy_constructor)

        argument list must satisfy all following conditions

        1.  given the class type **T**, the argument list must be &amp;&amp; like.
        2.  it will be  T&amp;&amp;, const T &amp;&amp;, volatile T&amp;&amp;, const volatile T&amp;&amp;.

        When to call the move constructor?

        -   initialization like T a = std::move(b)
        -   argument passing f(std::move(a)
        -   return type like return a T f()

        [Move Assignment Ref Detail](https://en.cppreference.com/w/cpp/language/move_assignment)

        like a = std::move(a2)
        The copy assignment operator is called whenever selected by overload resolution, e.g. when an object appears on the left side of an assignment expression.

        ```c++

        // An FDWrapper cannot be copied or moved
        FDWrapper( const FDWrapper& other ) = delete;
        // copy constructor
        FDWrapper& operator=( const FDWrapper& other ) = delete;
        // copy assignment
        FDWrapper( FDWrapper&& other ) = delete;
        // move constructor
        FDWrapper& operator=( FDWrapper&& other ) = delete;
        // move assignment
        ```


#### webget sequence {#webget-sequence}

the sequence of tcp connect is like this..

-   construct TCPSocket
-   construct Address
-   connect
-   write buffer to get url page
-   wait to read like string operation

    all these functions will call CheckSystemCall and then unix_error will call kernel api like socket connect and or so.

<!--listend-->

```c++

class unix_error : public tagged_error
{
public:
  explicit unix_error( const std::string_view s_attempt, const int s_errno = errno )
    : tagged_error( std::system_category(), s_attempt, s_errno )
  {}
};

```


### 3.2 byte stream {#3-dot-2-byte-stream}

In this section I will implement a pipe like byte stream, which a Writer puts data into the pipe and the Reader pops it.

After reading the check0.pdf, I firstly choose string as a ring buffer.

<!--list-separator-->

-  compile error and run error

    ```sh
    $ cmake --build build --target check0
    it will print out this message like that infinitely.
    AddressSanitizer:DEADLYSIGNAL
    AddressSanitizer:DEADLYSIGNAL
    AddressSanitizer:DEADLYSIGNAL
    AddressSanitizer:DEADLYSIGNAL
    AddressSanitizer:DEADLYSIGNAL
    AddressSanitizer:DEADLYSIGNAL
    AddressSanitizer:DEADLYSIGNAL
    ```

    After that I search from StackOverflow, it turns out that the linux setting is not suitable for clang. The solution for this is

    ```sh
    $sudo sysctl vm.mmap_rhd_bits=28(check the value by sudo sysctl vm.mmap_rhd_bits)
    after reboot this change will disappear. And if I wanna pernament change this setting, I have to change the etc file of /etc/sysctl.conf
    add `vm.mmap_rhd_bits=28` in this file
    $sudo sysctl -p
    if not just reboot
    ```

<!--list-separator-->

-  std::string_view

    The std::string_view class I don't fully understand, so I failed in peek tests. That is a ring buffer is wrapped around when start and end pointers are not equal, and string_view class needs a sequence char in memory. So i fail in the peek tests.

    After that, I switch to deque/queue. In this way, I passed 7 tests, but stuck in 8 peek tests,
    ![](/cs144/images/lab0_c++_error_heap_overflow.png)

    it says that heap buffer overflow when I didn't use gdb in eshell. The reason why i got this error is still string_view needs a sequence capture char in memory. And deque/queue implementation is chunks of memory linked, each set of memory is in sequence but not all the items. I use gdb to prove that.
    ![](/cs144/images/lab0_c++_error_deque.png)
    ![](/cs144/images/lab0_c++_deque_2.png)

    From the picture, we can see that the deque has discontinuous memory address the same as queue, that's why I get this error for several days to handle.

    Then I switched to list vector as my pipe container. Everything works great, except that string_view must constructs with continuous iterator, we all knew that linked-list has discontinous items when inset and remove.

    The final approach to handle peek function is use another string whenever I push or pop from the pipe I always rearrange the pipe so that it is in sequential memory using **copy** method.

    ```c++

    void Writer::push( string data )
    {
      // ........

      // reflush the pipe_view after each push and pop
      if(pipe_e > pipe_s){
        pipe_.copy(&pipe_view[0], pipe_.size(), pipe_s);
      }else{
        pipe_.copy(&pipe_view[0], capacity_ - pipe_s, pipe_s);
        pipe_.copy(&pipe_view[capacity_ - pipe_s], pipe_e, 0);
      }
    }

    void Reader::pop( uint64_t len )
    {
      // ....

      // reflush the pipe_view after each push and pop
      if(pipe_e > pipe_s){
        pipe_.copy(&pipe_view[0], pipe_.size(), pipe_s);
      }else{
        pipe_.copy(&pipe_view[0], capacity_ - pipe_s, pipe_s);
        pipe_.copy(&pipe_view[capacity_ - pipe_s], pipe_e, 0);
      }
    }

    ```

    and if I do some changes in Peek function it will cause compilation error like that.

    ```err
    error: invalid conversion from ‘const __gnu_cxx::__alloc_traits<std::allocator<char>, char>::value_type*’ {aka ‘const char*’} to ‘char*’ [-fpermissive]
    ```

    That is because Peek function is a const member function which has no right to change the member value.

    After adopting this way, I finally successfully passed the tests, though the speed is not so fast but i give in my efforts. There are a lot of efforts to take to make the code looks great and faster. I will do it in the future.
    ![](/cs144/images/lab0_c++_successfully.png)

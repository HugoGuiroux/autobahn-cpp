# **Autobahn**|Cpp

**Autobahn**|Cpp is a subproject of [Autobahn](http://autobahn.ws/) which provides a [WAMP](http://wamp.ws/) implementation in C++ with the following roles

 * **Caller**
 * **Callee**
 * **Publisher**
 * **Subscriber**

running over TCP(-TLS), Unix domain sockets or pipes (`stdio`), using `rawsocket-msgpack` WAMP transport.

The API and implementation make use of modern C++ 11 and `std::future`. Here is some code

```c++
// 1) call a remote procedure
//
auto c1 = session.call("com.mathservice.add2", {23, 777}).then(
   [&](boost::future<boost::any> f) {

      // call result received
		//
		std::cout << "Got RPC result " << any_cast<uint64_t> (f.get()) << std::endl;
	});
);

// 3) publish an event to a topic
//
session.publish("com.myapp.topic2", {23, true, std::string("hello")});

// 4) subscribe an event handler to a topic
//
auto s1 = session.subscribe("com.myapp.topic1",
   [](const anyvec& args, const anymap& kwargs) {
      cerr << "Got event: " << any_cast<uint64_t>(args[0]) << endl;
   }).then(
   [](future<subscription> sub) {
      cerr << "Subscribed with subscription ID " << sub.get().id << endl;
   });

```

The library is "header-only", light-weight (< 2k code lines) and **depends on** the following:

 * C++ 11 compiler
 * `boost::future`
 * `boost::any`
 * `boost::asio`
 
The library and example programs are developed with

 * clang 3.4
 * libc++


> Notes:
>
> * Support for GNU g++/libstdc++ depends on this [issue](https://github.com/tavendo/AutobahnCpp/issues/1)
> * While C++ 11 provides a `std::future` - but this does not yet support continuations. **Autobahn**|Cpp makes use of `boost::future.then` for attaching continuations to futures as outlined in the proposal [here](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2013/n3634.pdf). This feature will come to standard C++, but probably not before 2015 (see [C++ Standardisation Roadmap](http://isocpp.org/std/status))
> * Support for `when_all` and `when_any` depends on Boost 1.56 (!) or higher.


## Building

### Build tools

Install some libs and build tools:

```shell
sudo apt-get install libbz2-dev libssl-dev ruby libtool autoconf scons
```

### clang

Install [clang](http://clang.llvm.org/) and [libc++](http://libcxx.llvm.org/):

```shell
sudo apt-get install clang-3.4 libc++1 libc++-dev
```

### Boost

Get the latest Boost from [here](http://www.boost.org/). Then

```shell
cd $HOME
tar xvjf Downloads/boost_1_55_0.tar.bz2
cd boost_1_55_0/
./bootstrap.sh
./b2 toolset=clang cxxflags="-stdlib=libc++" linkflags="-stdlib=libc++"
```

Get Boost trunk by doing:

```shell
git clone --recursive git@github.com:boostorg/boost.git


svn co  http://svn.boost.org/svn/boost/trunk boost-trunk
```


Add the following to `$HOME/.profile`

```shell
export LD_LIBRARY_PATH=${HOME}/boost_1_55_0/stage/lib:${LD_LIBRARY_PATH}
```

### MsgPack-C

Get [MsgPack-C](https://github.com/msgpack/msgpack-c) and build with clang:

```shell
cd $HOME
git clone https://github.com/msgpack/msgpack-c.git
cd msgpack-c
./bootstrap
CXX=`which clang++` CC=`which clang` CXXFLAGS="-std=c++11 -stdlib=libc++" \
   LDFLAGS="-stdlib=libc++" ./configure --prefix=$HOME/msgpack_clang
make
make install
```

Add the following to `$HOME/.profile`

```shell
export LD_LIBRARY_PATH=${HOME}/msgpack_clang/lib:${LD_LIBRARY_PATH}
```

### **Autobahn**|Cpp

Finally, to build **Autobahn**|Cpp

```shell
source $HOME/.profile
cd $HOME
git clone git@github.com:tavendo/AutobahnCpp.git
cd AutobahnCpp
scons
```

## Building Documentation

The documentation is built using Sphinx, Doxygen and Breathe.

Doxygen takes the C++ source files and autogenerates XML files from the documented C++ source code.

Breathe takes the XML files generated by Doxygen and generates Sphinx RST files.

Sphinx takes the RST files generated plus manually written RST files and generates the final documentation in HTML format.

### Install documentation build tools

```shell
sudo apt-get install doxygen python-sphinx
sudo /usr/bin/pip install breathe
```


_gen/doxygen



## Futures

* [ASIO C++11 Examples](http://www.boost.org/doc/libs/1_55_0/doc/html/boost_asio/examples/cpp11_examples.html)
* [Using Asio with C++11](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3388.pdf)
* [C++17: I See a Monad in Your Future! ](http://bartoszmilewski.com/2014/02/26/c17-i-see-a-monad-in-your-future/)
* [Boost Thread](http://www.boost.org/doc/libs/1_55_0/doc/html/thread.html)
* [Boost Issue: when_all](https://svn.boost.org/trac/boost/ticket/7447)
* [Boost Issue. when_any](https://svn.boost.org/trac/boost/ticket/7446)
* [Boost Issue: future fires twice](https://svn.boost.org/trac/boost/ticket/9711)
* [Boost C++ 1y](http://www.boost.org/doc/libs/1_55_0/doc/html/thread/compliance.html#thread.compliance.cxx1y.async)

## Closures Cheetsheet

* `[]` Capture nothing (or, a scorched earth strategy?)
* `[&]` Capture any referenced variable by reference
* `[=]` Capture any referenced variable by making a copy
* `[=, &foo]` Capture any referenced variable by making a copy, but capture variable `foo` by reference
* `[bar]` Capture `bar` by making a copy; don't copy anything else
* `[this]` Capture the this pointer of the enclosing class

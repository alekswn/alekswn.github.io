---
layout: post
title: "In-place profiling in c++11: Part 1: Timing"
---

This tutorial shows how to implement a tiny robust benchmarking tool in C++11. 
This tool might be helpful for CS students as easy benchmark for algorithmic tasks and for real projects as a template for build-in profiler of mission critical methods.

* TOC
{:toc}

## Getting started

C++11 standard provides two types of real-time clocks :  old-fascinate _std::clock\_t_  and brand new _std::chrono_ interface. I chose _std::chrono::high\_resolution\_clock_ for this example. 

The idea is to create a single object controlling the timer  in the scope of benchmarked portion of code. For example, if  we'll create an object that starts timers at the beginning of the function it is destroyed automatically at the end of the function.
So we can place the stats logging facilities into the destructor of our class.

Let's start with the code. As a first step we'll declare _namespace BENCH11_  containing benchmarking class _Timer_ and logging class _Logger_. 

The _Timer_ class inteface:
{% highlight c++ %}
#include <string>
#include <chrono>
{% endhighlight %}

{% highlight c++ %}
namespace BENCH11 {    
    class Timer {
        const Logger logger;
        const std::chrono::high_resolution_clock::time_point t_start
    
    public: 
        Timer(std::string methodName, std::string fileName, int lineNumber);
        ~Timer();

    };
}
{% endhighlight %}
As you can see, the _Timer_ class  does not have any public methods besides the constructor and the destructor. It has fields to keep starting values of the timers and encapsulates a _Logger_ object. The constructor has a set of arguments to identify the place in the code where it was called. 

Now we're going to defile the interface of the _Logger class_:
{% highlight c++ %}
#include <string>
{% endhighlight %}
{% highlight c++ %}
namespace BENCH11 {
    class Logger {
    public:
        void logMessage(const std::string& message) const;
        Logger(std::string methodName, std::string fileName, int lineNumber);
    };
}
{% endhighlight %}
Here are just one public method _logMessage()_ and a public constructor.

Now it's a time for a black magic of defines. Let's say we do not need benchmarking in a release build and we do not want to pass all these boring arguments to the constructor.

So let's make a couple of **#define**s :
{% highlight c++ %}
#ifdef BENCH11
#define BENCH11_TIMER BENCH11::Timer BEHCH11_TIMER_OBJ(__PRETTY_FUNCTION__, __FILE__, __LINE__);
#else
#define BENCH11_TIMER
#endif
{% endhighlight %}
If the macro **BENCH11** was defined the macro **BENCH11\_TIMER** is creating an object of _BENCH11::Timer_ class with appropriate source code references. If the  macro **BENCH11** was not defined (in a release build for example)  the macro **BENCH11\_TIMER** is empty.
With these defines it's enough to place **BENCH11\_TIMER** macro to the very beginning of the profiled method or scope. See a usage example below:
{% highlight c++ %}
#include <limits>

#define BENCH11
#include "bench11.h"

void worker() {
    BENCH11_TIMER;
    for (int i = 0; i < std::numeric_limits<int>::max()/10; i++) {
         if (i == 10) {
             BENCH11_TIMER;
         }
    }
}

int main() {
    worker();
}
{% endhighlight %}
Here are two nested benchmarks inside the _worker()_ method.

## _Timer_ class implementation

Implementation of _Timer_ is pretty straightforward. In the constructor we store the current  count of a system timer and initialize the _Logger_. In the destructor we calculate the difference between the moments of construction and destruction and pass a message to the _Logger_. I'd place these routines right into the declaration section.
{% highlight c++ %}
#include <string>
#include <sstream>
#include <chrono>
{% endhighlight %}
{% highlight c++ %}
namespace BENCH11 {    
    class Timer {
        const Logger logger;
        const std::chrono::high_resolution_clock::time_point t_start;    
    public:
        Timer(std::string methodName, std::string fileName, int lineNumber) 
        :logger(methodName, fileName, lineNumber),
         t_start(std::chrono::high_resolution_clock::now())
        {}
        ~Timer() {
            auto t_end = std::chrono::high_resolution_clock::now();
            std::basic_ostringstream<char> os;
            os   << "Wall time: "
                 << std::chrono::duration<double, std::milli>(t_end-t_start).count()
                 << " ms";
            logger.logMessage(os.str());
        }
    };
}
{% endhighlight %}

## Simple implementation of _Logger_

Our simple logger will write messages to standard error stream. There are two important thinks to do. The first is a tagging messages with the location in the source code. And the second issue we have to handle is I/O delays. We'll do output asynchronously using C++ threads. I'd place the implementation right into the declaration again:
{% highlight c++ %}
#include <string>
#include <iostream>
#include <thread>
{% endhighlight %}
{% highlight c++ %}
namespace BENCH11 {
    class Logger {
        std::string TAG = "[ BENCH11 ]: ";
    protected:
        static void logMessageSync(const std::string& message) {
            std::cerr << message << std::endl;
        }
        static void logMessageAsync(const std::string& message) {
            std::thread tr(logMessageSync, message);
            tr.detach();
        }
    public:
        void logMessage(const std::string& message) const {
            logMessageAsync(TAG + message); 
        }

        Logger(std::string methodName, std::string fileName, int lineNumber) {
            TAG.append("(");
            TAG.append(methodName);
            TAG.append(":");
            TAG.append(fileName);
            TAG.append(":");
            TAG.append(std::to_string(lineNumber));
            TAG.append(") ");
        }
    };
}
{% endhighlight %}
Here we have a new field string _TAG_ and two helper methods. _TAG_ is filled in in the constructor and is utilized as a prefix to the message by the _logMessage()_ method. _logMessage()_ calls the static helper method _logMessageAsync()_. This method creates a detached thread for the static worker method _logMessageSync()_ which performs real I/O operations.  

## Building

This example might be build using C++11 or higher standard. Successful linkage require threading support.
For  *gcc* compiler command line will look like that:
{% highlight console %}
g++ --std=c++11 -lpthread -I../include/ test_bench.cpp  -o test_bench
{% endhighlight %}

## Future improvements

There are a great variety of possible improvements according to the usage purposes. As far as the presented realization relays on the wall but not on the processor's it is not suitable for benchmarking methods with I/O calls and might give wrong results if there were high CPU load from the background processes. 
Adding _std::clock()_ timer will help to handle these issues.

There are a lot of ways to implement more reliable _Logger_. For example, it could write to a file or network socket. Also the primitive concurrency model implemented in this example might mix up output from different _Logger_ instances. Therefore it would be better to implement a single output thread with a message queue.

Despite of the facts above this example is a good starting point.  

## References

You can find the full updated sources of this example and much more at my [GitHub page](https://github.com/alekswn/Bench11/tree/master).

Stay tuned  for the **Part 2: Memory and Leaks**

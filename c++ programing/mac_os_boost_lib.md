#Mac OS Boost Lib
C++ Code using Boost lib

```
#include <boost/thread/thread.hpp>
#include <boost/lockfree/queue.hpp>
#include <iostream>

#include <boost/atomic.hpp>

boost::atomic_int producer_count(0);
boost::atomic_int consumer_count(0);

boost::lockfree::queue<int> queue(128);

const int iterations = 10000000;
const int producer_thread_count = 4;
const int consumer_thread_count = 4;

void producer(void)
{
    for (int i = 0; i != iterations; ++i) {
        int value = ++producer_count;
        while (!queue.push(value))
            ;
    }
}

boost::atomic<bool> done (false);
void consumer(void)
{
    int value;
    while (!done) {
        while (queue.pop(value))
            ++consumer_count;
    }

    while (queue.pop(value))
        ++consumer_count;
}

int main(int argc, char* argv[])
{
    using namespace std;
    cout << "boost::lockfree::queue is ";
    if (!queue.is_lock_free())
        cout << "not ";
    cout << "lockfree" << endl;

    boost::thread_group producer_threads, consumer_threads;

    for (int i = 0; i != producer_thread_count; ++i)
        producer_threads.create_thread(producer);

    for (int i = 0; i != consumer_thread_count; ++i)
        consumer_threads.create_thread(consumer);

    producer_threads.join_all();
    done = true;

    consumer_threads.join_all();

    cout << "produced " << producer_count << " objects." << endl;
    cout << "consumed " << consumer_count << " objects." << endl;
}

```
## Install Boost for mac

1. download boost for mac os

	`brew install boost`
2. check the path of boost lib

	`brew list boost`


## Eclipse CDT

1. Create a project in Eclipse CDT	
2. Create cpp file which contains the code above.
3. Open Project Properties (`ctrl+i`)
4. Choose C/C++ General -> Path and Symbols -> includes tab -> GNU C++, and then add..., add the path of boost include dir. Here might be '/usr/local/Cellar/boost/1.57.0/include/'
5. Choose Libraries tab, and then add libararies like: boost_filesystem, boost_thread-mt, boost_system, you can refer to the /usr/local/Cellar/boost/1.57.0/lib for details.
	
## Compile & Run
### Compile
`ctrl + b` compile the the project. and the binary file will appear, if there is no error at all.

### Run

1. Highlight the project folder, right click and select "Properties"
2. Select the "Run/Debug Settings" tab
3. You should see "test_set" in the list. Highlight it and click "Edit" on the
   right-hand side. 
4. When the window pops up, open the "Environment" tab
5. Click "New" on the right-hand side.
6. In the "Name:" space, enter "DYLD_LIBRARY_PATH"
7. In the "Value:" space, "/usr/local/Cellar/boost/1.57.0/lib "
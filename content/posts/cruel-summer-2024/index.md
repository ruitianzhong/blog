---
title: "Cruel Summer in 2024"
subtitle: "To record what I did in the summer of 2024"

summary: "To record what I did in the summer of 2024"

date: 2024-06-30T16:40:00+08:00

lastmod: 2024-08-30T17:03:00+08:00

draft: false


# show_date: true
# links:
#   - icon_pack: fab
#     icon: twitter
#     name: Follow
#     url: 'https://twitter.com/Twitter'
#   - icon_pack: fab
#     icon: medium
#     name: Originally published on Medium
#     url: 'https://medium.com'
featured: false
authors:
  - admin

categories:
- daily-record
---


## 9.6-now

+ Read *RUDRA: Finding Memory Safety Bugs in Rust at the Ecosystem Scale*
+ Read *OCOLOS: Online COde Layout OptimizationS*

## 9.1-9.5

+ Solve LeetCode problem

## 8.30

Error handling in C++

```cpp
#include <iostream>
#include <utility>
int divide(int a, int b) {
  if (b == 0)
    throw "Divide by zero";
  return a / b;
}

int main() {
  try {
    divide(1, 0);

  } catch (const char *msg) {
    std::cout << msg << std::endl;
  }
  try {
    divide(10, 0);
  } catch (...) {
    using namespace std;
    cout << "handling error" << endl;
  }
  return 0;
}

```

Error handling in Go

```go
package main

import "fmt"

func main() {

        test()

}

func test() {
        defer func() {
                if err := recover(); err != nil {

                        fmt.Println(err)
                }
        }()

        panic("Some err")

}

```

`decltype` and `auto` in C++

```cpp
#include <iostream>
#include <tuple>
#include <utility>
#include <vector>
auto add(int x, int y) -> decltype(x + y) { return x + y; }
using namespace std;

auto id(int &&x) -> int {
  cout << "rv ref" << endl;
  return x;
}

auto id(int &x) -> int {
  cout << "lv ref" << endl;
  return x;
}

auto sub(int x, int y) { return x - y; }
int main() {
  cout << add(1, 1) << endl;
  int i{0};
  id(i++);
  id(++i);
  const int x = 1;
  auto y = x;
  y++; // no problem
  auto init = {1, 2, 3, 4, 5};

  vector<int> v = init;
  sub(1, 1);

  pair<int, int> p = {1, 1};
  cout << p.first << " " << p.second << endl;
  auto [a, b] = p;
  int t1, t2;
  tuple tp = {1, 2};
  cout << a << " " << b << endl;
  tie(t1, t2) = tp;
  cout << t1 << " " << t2 << endl;
  tie(t1, t2) = p;
  cout << t1 << " " << t2 << endl;

  return 0;
}
```

Perfect forwarding in C++

```cpp
#include <iostream>
#include <utility>
using namespace std;

template <typename T> void test(T &&t) { cout << "rvalue ref" << endl; }

template <typename T> void test(T &t) { cout << "lvalue ref" << endl; }

template <typename T> void forwardTest(T &&t) {
  test(t);
  test(std::move(t));
  test(std::forward<T>(t));
  cout << "---end---" << endl;
}

int main() {
  auto &&v1 = int(0);
  // universal reference
  int x = 1;
  forwardTest(x);
  forwardTest(int(42));
  return 0;
}
```

Lambda and constexpr in C++

```cpp
#include <iostream>
using namespace std;
#include <algorithm>
#include <utility>
constexpr auto first = 42;
struct Data {
public:
  int x;
  int y;
  Data(int _x, int _y) : x(_x), y(_y) {}
};
// Example for constexpr and lambda (value, reference and std::move)
// Something about the Person.
constexpr int add(int x, int y) { return x + y; }

struct Person {
  int height;
  Person(int h) : height(h) {}
};
int main(int argc, char *argv[]) {

  int i = 0;
  auto f1 = [&i](int x) -> int {
    i += x;
    return 42;
  };

  f1(1);
  f1(1);
  // non-static variable can be captrued
  vector<Data> v1;

  v1.emplace_back(1, 2);
  v1.emplace_back(3, 2);
  v1.emplace_back(1, 3);
  sort(v1.begin(), v1.end(), [](Data d1, Data d2) {
    if (d1.x > d2.x) {
      return false;
    }
    if (d1.x < d2.x) {
      return true;
    }
    return d1.x < d2.y;
  });
  for (auto d : v1) {
    cout << d.x << " " << d.y << endl;
  }
  vector<Person> v2;

  v2.emplace_back(178);
  v2.emplace_back(160);
  v2.emplace_back(159);

  sort(v2.begin(), v2.end(),
       [](Person a, Person b) { return a.height < b.height; });
  for (auto &p : v2) {
    cout << p.height << endl;
  }

  vector<Person> v3 = move(v2);
  cout << v2.size() << endl;
  // moved into lambda expression here?
  auto f2 = [v4 = std::move(v3)]() {
    cout << v4.size() << endl;
    return;
  };
  f2();
  cout << first << endl;
  cout << add(1, 1) << " " << add(v3.size(), argc) << endl;
  array<int, add(1, 1)> arr;
  // constexpr in constructor function??
  // not allow!  first=1000;
  return 0;
}
```

```bash
echo ${!map1[@]}
for key in ${!map1[@]};do
echo $key
done
```

```go
package main

import (
        "fmt"
        "math/rand"
        "sync"
        "time"
)

func main() {

        var wg sync.WaitGroup
        ch := make(chan Data, 5)
        wg.Add(2)
        go produce(&wg, ch)
        go consume(&wg, ch)
        wg.Wait()

        ch1 := make(chan Data)

        select {
        case d := <-ch1:
                fmt.Println(d)

        case <-time.After(1):
                fmt.Println("timeout")

        }

        close(ch1)
        // ok的作用？？？
        _, ok := <-ch1
        fmt.Println(ok)
        multiChannel()
        ch3 := make(chan int)
        close(ch3)
        <-ch3
        ch4 := make(chan int)

        select {
        case ch4 <- 1:
                fmt.Println("ch4")
        default:
                fmt.Println("default")
        }

}

func send(ch chan<- int, num int) {

        for i := 0; i < 10; i++ {
                ch <- num + i
                t := time.Duration(rand.Int() % 800)
                time.Sleep(t * time.Millisecond)
        }
        close(ch)

}

func multiChannel() {

        ch1 := make(chan int)
        ch2 := make(chan int)
        go send(ch1, 1)
        go send(ch2, 2)
        // cnt:=0

        for i := 0; i < 20; i++ {

                select {
                case x, ok := <-ch1:
                        fmt.Println(x, 1, ok)

                case x, ok := <-ch2:
                        fmt.Println(x, 2, ok)

                }

        }

}

type Data struct {
}

func produce(wg *sync.WaitGroup, ch chan<- Data) {
        for i := 0; i < 10; i++ {
                ch <- Data{}
        }
        close(ch)
        wg.Done()
}

func consume(wg *sync.WaitGroup, ch <-chan Data) {
        for _ = range ch {
                fmt.Println("consuming")
        }
        wg.Done()
}
```

## 8.28-8.29

+ Interview...

## 8.9-8.24

+ Solve Leetcode problems...(Hot 100 and Hot 150)

## 8.8

+ Solve Leetcode problems...

## 8.7

+ Read [Thinking outside the box: My PhD Odyssey From Single-Server Architecture to Distributed Datastores (Part 1)](https://www.sigops.org/2024/thinking-outside-the-box-my-phd-odyssey-from-single-server-architecture-to-distributed-datastores/) and [Thinking outside the box: My PhD Odyssey From Single-Server Architecture to Distributed Datastores (Part 2)](https://www.sigops.org/2024/thinking-outside-the-box-my-phd-odyssey-from-single-server-architecture-to-distributed-datastores-part-2/)

## 8.6

+ Read [Advice for Undergraduate](https://cs.stanford.edu/people/karpathy/advice.html)
+ Watch video about [Graph Neural Network(GNN)](https://www.bilibili.com/video/BV1iT4y1d7zP)
+ Read [Revisiting Distributed Memory in the CXL Era](https://www.sigops.org/2024/revisiting-distributed-memory-in-the-cxl-era/)

+ Something about Golang's interface

```go
package main

import "fmt"

type Animal interface {
 run()
 skip()
}

type Cat struct {
 x int
}

func (c Cat) run() {
 fmt.Println("this is cat")
 c.x = 0
}

// not a good practice
func (c *Cat) skip() {
 fmt.Println("cat skip1!")
 for i, v := range "hello world" {
  fmt.Println(i, v)
 }
}

func main() {
 c := Cat{x: 42}
 c.run()
 var a Animal
 fmt.Println(a == nil)

 a = &c
 // a = c is not ok
 a.run()
 fmt.Println(c.x)
}

```

### 8.5

+ Learn how to use snort

```bash
sudo ./snort -vd -i <interface_name> "port 9999"
curl http://localhost --connect-timeout 2000
mkdir dir
```

+ Learn more about [bash](https://liujiacai.net/blog/2024/04/05/robust-shell-scripting). Useful material: [Shell Style Guide from Google](https://google.github.io/styleguide/shellguide.html)

```bash
if [[ "foo" == "f*" ]] ; then echo hello; else  echo test; fi
# blank is important
if [[ "foo" == f* ]] ; then echo hello ; else echo test; fi
for (( i=0;i<5;i++)) ; do echo hello ; done
# Here document
 cat <<EOF                               
`ls`
EOF
# (())
if [ $((1+1)) -eq 2 ] ; then echo yes ; fi

# Why export?
x=1
export y=2
# case statement
case $p in
world)
echo hello 
;;
esac

declare -a arr=(a b c)
declare -a map=([hello]=world)
echo "${arr[@]}"
echo "${arr}"

echo $$
echo $@
echo $?

. example.sh
```

+ Learn about the implication of batch size

### 8.4

+ Read something about diffusion model

> A note about promiscuous vs. non-promiscuous sniffing: The two techniques are very different in style. In standard, non-promiscuous sniffing, a host is sniffing only traffic that is directly related to it. Only traffic to, from, or routed through the host will be picked up by the sniffer. Promiscuous mode, on the other hand, sniffs all traffic on the wire. In a non-switched environment, this could be all network traffic. The obvious advantage to this is that it provides more packets for sniffing, which may or may not be helpful depending on the reason you are sniffing the network. However, there are regressions. Promiscuous mode sniffing is detectable; a host can test with strong reliability to determine if another host is doing promiscuous sniffing. Second, it only works in a non-switched environment (such as a hub, or a switch that is being ARP flooded). Third, on high traffic networks, the host can become quite taxed for system resources.

+ `perf` usage notes

```bash
taskset --cpu-list 2 ls # 2 is not a mask
taskset -p pid
```

+ Learn about isolate certain cpu cores. [link](https://unix.stackexchange.com/questions/326579/how-to-ensure-exclusive-cpu-availability-for-a-running-process/326585#326585)
+ Read [A Brief History of JavaScript Frameworks](https://primalskill.blog/a-brief-history-of-javascript-frameworks)

### 8.3

+ Read something about reinforcement learning and GAN
+ Read [Pytorch: An imperative style, high-performance deep learning library](https://arxiv.org/abs/1912.01703)

### 8.1-8.2

+ Back to Xi'an
+ Read Matrix Calculus

### 7.31

+ Read [PROGRAMMING WITH PCAP](https://www.tcpdump.org/pcap.html)

### 7.30

+ Read [计算机系统会议论文是如何评审的](https://zhaoxiahust.github.io/blog/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%B3%BB%E7%BB%9F%E4%BC%9A%E8%AE%AE%E8%AE%BA%E6%96%87%E6%98%AF%E5%A6%82%E4%BD%95%E8%AF%84%E5%AE%A1%E7%9A%84.pdf) written by Haibo Chen

### 7.29

+ Read [Mapping High Level Constructs to LLVM IR](https://mapping-high-level-constructs-to-llvm-ir.readthedocs.io/en/latest/basic-constructs/local-variables.html)

### 7.26-7.27

+ Read [Kubernetes Tutorials](https://kubernetes.io/docs/tutorials/)

### 7.25

+ Watch video about [Flash Attention](https://www.bilibili.com/video/BV1UT421k7rA)

### 7.24

+ Watch [Kaiming He's Talk @ CUHK](https://www.bilibili.com/video/BV1Zz421B7Vf)

### 7.23

+ Read [The Ultimate Guide to Kubernetes Networking and Multi-Tenant Gateways](https://blog.moelove.info/the-ultimate-guide-to-kubernetes-networking-and-multi-tenant-gateways)
+ Watch video about [Automatic Mixed Precision](https://www.bilibili.com/video/BV1qJ4m1w7ur)
+ Read [Vector Database](https://guangzhengli.com/blog/zh/vector-database/)

### 7.22

+ Read [Hardware Accelerator (CUDA, DSA, etc.)](https://openmlsys.github.io/chapter_accelerator/index.html)
+ Watch video about [KV Cache, Page Attention](https://www.bilibili.com/video/BV1kx4y1x7bu)
+ Watch video about [Distributed Training(DeepSpeed)](https://www.bilibili.com/video/BV1mm42137X8/)
+ Watch video about [Reinforcement Learning: PPO](https://www.bilibili.com/video/BV1iz421h7gb)
+ Watch video about [GPU (Tensor Core)](https://www.bilibili.com/video/BV1rH4y1c7Zs) SIMT, Warp(32 threads), Streaming Multiprocessor, CUDA core(more general) VS. Tensor Core(add and multiply in a period), Special Function Unit, Compute Intensity
+ Watch [Kaiming He's Talk on Learning Deep Representation](https://www.bilibili.com/video/BV1sW421c7SK) (really insightful talk！)

### 7.21

+ Read [Distributed Training](https://openmlsys.github.io/chapter_distributed_training/index.html) (Pipeline Parallelisim，Model Parallelism，Parameter Server，Collective Communication)
+ Read [Programming Interface](https://openmlsys.github.io/chapter_programming_interface/index.html) (Python & C++)
+ Read [Computation Graph](https://openmlsys.github.io/chapter_computational_graph/index.html) in ML framework

> 一类是学习率不受梯度影响的随机梯度下降（Stochastic Gradient Descent）及SGD的一些改进方法，如带有Momentum的SGD；另一类是自适应学习率如AdaGrad、RMSProp、Adam等。

### 7.20

+ Read [Python Numpy Tutorial](https://cs231n.github.io/python-numpy-tutorial/)
+ Read [Gradient Check](https://cs231n.github.io/neural-networks-3/)
+ Read [Image Similarity Search and Cosine Similarity](https://www.trybackprop.com/blog/linalg101/part_3_build_image_search)
+ Find [GNN Course](http://web.stanford.edu/class/cs224w/) and [Deep Reinforcement Learning](https://rail.eecs.berkeley.edu/deeprlcourse/)

### 7.19

+ Read [Visualize CNN](https://cs231n.github.io/understanding-cnn/)
+ Read [SGD](https://cs231n.github.io/optimization-1/)
+ Read [Simple Neural Network](https://cs231n.github.io/neural-networks-case-study/)

### 7.18

+ Read [Learning Research](https://github.com/pengsida/learning_research)

### 7.17

+ Read [Generative Model(Autoregressive Model & VAE & GAN)](https://cs231n.stanford.edu/slides/2024/lecture_13.pdf)
+ Read [RNN](https://cs231n.stanford.edu/slides/2024/lecture_7.pdf)

### 7.16

+ Read [Transfer Learning, Batch Normalization,Case Study on AlexNet, VGG, GoogleNet and ResNet](https://cs231n.stanford.edu/slides/2024/lecture_6_part_1.pdf)
+ Read [Why Momentum Really Works](https://distill.pub/2017/momentum/)
+ Read [How to apply for CVE](https://blog.csdn.net/m0_43397796/article/details/130985872)
+ Read [Training in Practice](https://cs231n.stanford.edu/slides/2024/lecture_6_part_2.pdf)
+ Read [Object Detection and Semantic Segmantation](https://cs231n.stanford.edu/slides/2024/lecture_9.pdf)
+ Why 'zero centered ' is better?

### 7.11-7.15

+ Interview...
+ Review Linear Algebra & Probability & Advanced Mathematic

### 7.8

+ Read [Classification](https://cs231n.github.io/classification/)
+ Read [Linear Classification](https://cs231n.github.io/linear-classify/)

### 7.6-7.7

+ Finish [dynamic linker lab](https://github.com/ruitianzhong/dynamic-linker)

### 7.5

+ Finish [static linker lab](https://github.com/ruitianzhong/linker)

### 7.4

+ Interview...

### 7.2

+ Read [Convolutional Neural Networks (CNNs / ConvNets)](https://cs231n.github.io/convolutional-networks/)
  
### 7.1

+ Read [Attention? Attention!](https://lilianweng.github.io/posts/2018-06-24-attention/)
+ Read [Derivatives, Backpropagation, and Vectorization](https://cs231n.stanford.edu/handouts/derivatives.pdf)

### 6.30

+ Read [Understanding LSTM Networks](https://colah.github.io/posts/2015-08-Understanding-LSTMs/)
+ Read [Back Propagation](http://cs231n.github.io/optimization-2)

### 6.29

+ Read [The Illustrated Transformer](http://jalammar.github.io/illustrated-transformer/)
+ Read [The Illustrated BERT](https://jalammar.github.io/illustrated-bert/)
+ Read [Visual and Interactive Guide to the Basics of Neural Networks](https://jalammar.github.io/visual-interactive-guide-basics-neural-networks/)

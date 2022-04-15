**A car is-a vehicle**

**A car has-a steering wheel**

```c++
class SteeringWheel
{};

class Vehicle
{
    virtual void doStuff() = 0;
};

class Car: public Vehicle // 记住是共有继承，其它继承不可以
{
    SteeringWheel  sWheel; // 作为成员变量
    virtual void doStuff();
};

Car *c = new Car;
Vehicle *p = c; // is-a
```

那么 private 和 protected 继承什么时候会用到呢？参见 https://www.zhihu.com/question/425852397/answer/1528656579

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


# 有限状态机 arduino 库

简称状态机,表示“有限个“状态以及在这些状态之间进行转换 Transation 和动作 Action 等行为的数学模型

# 文档

状态机的英文讲解：https://en.wikipedia.org/wiki/Finite-state_machine

状态机的中文讲解：https://www.jianshu.com/p/7690b207ae92

# 设计

库由两个类组成：Fsm 和 State

State：表示状态机中的状态。一个状态有两个回调函数与之关联： on_enter 和 on_exit， 分别在状态进入和退出的时候按顺序被调用。

Fsm：表示状态机。添加转换使用 add_transition 函数。此函数接受指向两种状态的指针，一个是事件id，另一个是在转换发生时调用的回调函数。通过事件id调用触发器将调用转换，但仅当Fsm处于启动状态时，否则不会发生任何事情。 

# 教学
首先在代码文件最前面创建 Fsm 和 State 实例。


    #include <Fsm.h>

    #define FLIP_LIGHT_SWITCH 1
    Fsm fsm;
    State state_light_on(on_light_on_enter, &on_light_on_exit);
    State state_light_off(on_light_off_enter, &on_light_off_exit);

在 Setup() 函数添加状态转换

    void setup()
    {
        fsm.add_transition(&state_light_on, &state_light_off,
                           FLIP_LIGHT_SWITCH,
                           &on_trans_light_on_light_off);
        fsm.add_transition(&state_light_off, &state_light_on,
                           FLIP_LIGHT_SWITCH,
                           &on_trans_light_off_light_on);
    }

这个例子中添加了两个转换：

当事件 FLIP_LIGHT_SWITCH 被触发时从 light on 状态到 light off 状态。
当事件 FLIP_LIGHT_SWITCH 被触发时从 light off 状态到 light on 状态。
定义你的回调函数，如果不需要回调，传递 NULL 到函数即可。


    void on_light_on_enter()
    {
      // Entering LIGHT_ON state
    }

    void on_light_on_exit()
    {
      // Exiting LIGHT_ON state
    }

    void on_light_off_enter()
    {
      // Entering LIGHT_OFF state
    }

    void on_light_off_exit()
    {
      // Exiting LIGHT_OFF state
    }

    void on_trans_light_on_light_off()
    {
      // Transitioned from LIGHT_ON to LIGHT_OFF
    }

    void on_trans_light_off_light_on()
    {
      // Transitioned from LIGHT_OFF to LIGHT_ON
    }

最后，触发一个状态转换。


    fsm.trigger(FLIP_LIGHT_SWITCH); // 触发 FLIP_LIGHT_SWITCH 事件

# 示例

Arduino 使用状态机的多任务系统
Beginner Arduino 程序初学者往往聚焦于单个目标，比如点亮并闪烁LED或者让伺服电机旋转，一般使用延时来控制时间，如下例： 


    void setup() {
      pinMode(13, OUTPUT);
    }

    void loop() {
      digitalWrite(13, HIGH);
      delay(1000);
      digitalWrite(13, LOW);
      delay(1000);
    }

这种点亮并闪烁LED的单任务会很好工作。然而延时影响了多个任务同时运行，我们可以添加另一个LED让它与现有的LED一起闪烁...


    void setup() {
      pinMode(13, OUTPUT);
      pinMode(12, OUTPUT);
    }

    void loop() {
      digitalWrite(13, HIGH);
      digitalWrite(12, HIGH);
      delay(1000);
      digitalWrite(13, LOW);
      digitalWrite(12, LOW);
      delay(1000);
    }

...但是如果我们想让新添加的LED以不同的频率闪烁，我们会发现延时暂停了程序，阻止了其它任务运行。比如读取传感器数据、数学运算、引脚操作等在延时期间事实上都被挂起了。

毫秒级的多任务处理
millis 返回当前程序在Arduino 板开始运行以来的毫秒时间，当 LED 切换到 ON ，当前毫秒时间被存储，随着程序运行，当前时间与 LED ON 的时间差和预定义的时间间隔比较后，LED可以被关闭。下面是官方演示实例：


    const int ledPin =  13;
    int ledState = LOW;

    unsigned long previousMillis = 0;
    const long interval = 1000;

    void setup() {
      pinMode(ledPin, OUTPUT);
    }

    void loop() {
      // check to see if it's time to blink the LED; that is, if the
      // difference between the current time and last time you blinked
      // the LED is bigger than the interval at which you want to
      // blink the LED.
      unsigned long currentMillis = millis();

      if (currentMillis - previousMillis >= interval) {
        // save the last time you blinked the LED
        previousMillis = currentMillis;

        // if the LED is off turn it on and vice-versa:
        if (ledState == LOW) {
          ledState = HIGH;
        } else {
          ledState = LOW;
        }

        digitalWrite(ledPin, ledState);
      }
    }

对于闪烁 LED 来说，跟踪时间并且引入多任务状态会让工作变得很好。

使用有限状态机
它是一个概括一个机器状态的计算模型，可以引入进来使得机器从一种状态到另一种状态，但机器在同一时间只能有一种状态。

如果我们假设你程序中的每个任务代表一种机器状态，我们可以使用状态机来为我们管理转换。我们的程序中闪烁两个 LED ，我们需要两个状态机，每个状态机有两种状态：一个 LED ON ，一个 LED OFF。

Arduino-fsm 库支持定时转换，不需要自己管理时序，下面是arduino-fsm 和 add_timed_transition 方法的应用实例：


    #include <Fsm.h>

    #define LED1_PIN 10
    #define LED2_PIN 11

    void on_led1_on_enter() {
        digitalWrite(LED1_PIN, HIGH);
    }

    void on_led1_off_enter() {
        digitalWrite(LED1_PIN, LOW);
    }

    void on_led2_on_enter() {
        digitalWrite(LED2_PIN, HIGH);
    }

    void on_led2_off_enter() {
        digitalWrite(LED2_PIN, LOW);
    }

    State state_led1_on(&on_led1_on_enter, NULL);
    State state_led1_off(&on_led1_off_enter, NULL);

    State state_led2_on(&on_led2_on_enter, NULL);
    State state_led2_off(&on_led2_off_enter, NULL);

    Fsm fsm_led1(&state_led1_off);
    Fsm fsm_led2(&state_led2_off);

    void setup() {
        pinMode(LED1_PIN, OUTPUT);
        pinMode(LED2_PIN, OUTPUT);

        fsm_led1.add_timed_transition(&state_led1_off, &state_led1_on, 1000, NULL);
        fsm_led1.add_timed_transition(&state_led1_on, &state_led1_off, 3000, NULL);
        fsm_led2.add_timed_transition(&state_led2_off, &state_led2_on, 1000, NULL);
        fsm_led2.add_timed_transition(&state_led2_on, &state_led2_off, 2000, NULL);
    }

    void loop() {
        fsm_led1.check_timer();
        fsm_led2.check_timer();
    }

这个例子中有两个状态机对应每一个LED，添加了下面的定时转换：

一个转换： LED1 off 到 LED1 on 在 1s 之后；
一个转换： LED1 on 到 LED1 off 在 3s 之后；
一个转换： LED2 off 到 LED2 on 在 1s 之后；
一个转换： LED2 on 到 LED2 off 在 2s 之后；
两个状态机都开始于 LED off 状态。程序段 loop 针对每一个状态机调用 check_timer 函数检查所有的延时转换，如果间隔到了就执行转换。


详见示例及下面链接：

* [Humble Coder: Arduino finite state machine library][1]
* [Humble Coder: Arduino multitasking using a finite state machine][2]

[1]: http://www.humblecoder.com/arduino-finite-state-machine-library/
[2]: http://www.humblecoder.com/arduino-multitasking-using-finite-state-machines/

# 贡献

If you'd like to contribute to `arduino-fsm` please submit a pull-request on a
feature branch.

# 捐献

* Bitcoin: 1HnqohdK1d6gwDc7bT6LPPkmUFAXczEJKp

# 更新日志

**2.2.0 - 25/10/2017**

* Add `on_state()` handler to states
* New `run_machine()` method to invoke machine execution (includes a `check_timed_transitions()` call)
* New `timed_switchoff.ino` example sketch to ilustrate new `on_state()` and `run_machine()` funcionality
* Corrections:
 - `make_transition()` correctly initialices timed transitions start milliseconds (`make_transition()` is now a fsm method)
 - Initial state `on_enter()` handler is now correctly executed on fsm first run
 - Removed `Serial.println(now);` trace in _Fsm.cpp_
 - Correct initialization of `m_num_timed_transitions`
 

**2.1.0 - 21/11/2015**

* Add timed transitions

**2.0.0 - 03/09/2015**

* Remove AUTHORS files: too much hassle to maintain
* Add library.properties
* Add keywords.txt
* Remove name attribute from state
* Use int for transition event instead of string

**1.0.0 - 24/12/2013**

* Initial release.

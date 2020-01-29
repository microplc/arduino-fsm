有限状态机（简称状态机,表示“有限个“状态以及在这些状态之间进行转换 Transation 和动作 Action 等行为的数学模型） arduino 库

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

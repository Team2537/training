# Feedforward Control

Much like PID control, feedforward is another type of control system. Unlike PID control, feedforward control is
open-loop, meaning it doesn't rely on feedback from the system to control it. This makes feedforward control much simpler
than PID control, but it's also less accurate.

Feedforward control is essentially a way to predict how much power you need to apply to a mechanism to get it to move to a
certain way. This is done using a very simple equation:

\\( V(x) = K_v * x + K_a * \dot{x} + K_s * sign(x) \\)

Where:

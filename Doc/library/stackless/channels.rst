.. _channels:

*******************************************
Channels --- Communication between tasklets
*******************************************

A channel object is used for communication between tasklets.

By sending on a channel within a tasklet, another tasklet that is waiting to
receive on the channel is resumed.  If there is no waiting receiver, then the
sender is suspended.

By receiving on a channel within a tasklet, another tasklet that is waiting
on the channel to send is resumed.  If there is no waiting sender, then the
receiver is suspended.

.. class:: channel()

.. method:: channel.send(value)

   Send a value over the channel.  If no other tasklet is already receiving on
   the channel, the sender will be blocked.  Otherwise, the receiver will be
   activated immediately, and the sender is put at the end of the runnables
   list.
   
   Example - sending a value over a channel::
   
       >>> c = stackless.channel()
       >>> def sender(chan, value):
       ...     chan.send(value)
       ...
       >>> stackless.tasklet(sender)(5)
       >>> stackless.run()
       >>> print c.receive()
       5

.. method:: channel.receive()

   Receive a value over the channel.  If no other tasklet is already sending
   on the channel, the receiver will be blocked. Otherwise, the sender will be
   activated immediately, and the receiver is put at the end of the runnables
   list.
   
   Example - receiving a value over a channel::
   
       >>> c = stackless.channel()
       >>> def receiver(chan):
       ...     value = chan.receive()
       ...     print value
       ...
       >>> stackless.tasklet(receiver)(c)
       >>> stackless.run()
       >>> c.send(5)
       5

.. method:: channel.send_exception(exc, *args)

   Send an exception over the channel.  The behaviour is the same as for
   :meth:`send`, however the receiving tasklet has the exception raised
   on it rather than receiving a value as it expects.

   Example - sending an exception to a waiting receiver::

       >>> c = stackless.channel()
       >>> def receiver(chan):
       ...     chan.receive()
       ...
        >>> stackless.tasklet(receiver)(c)
        <stackless.tasklet object at 0x01BB8130>
        >>> stackless.run()
        >>> c.send_exception(Exception, "xxx")
        Traceback (most recent call last):
          File "<stdin>", line 1, in <module>
          File "<stdin>", line 2, in receiver
        Exception: xxx

.. method:: channel.send_sequence(seq)

   Send a stream of values over the channel.  Combined with a generator, this
   is a very efficient way to build fast pipes.

   Example - sending a sequence over a channel::
   
       >>> def sender(channel):
       ...     channel.send_sequence(sequence)
       ...
       >>> def receiver(channel):
       ...     count = 0
       ...     while count < len(sequence):
       ...             count += 1
       ...             value = channel.receive()
       ...             print value
       ...
       >>> c = stackless.channel()
       >>> stackless.tasklet(sender)(c)
       <stackless.tasklet object at 0x01BB84F0>
       >>> stackless.tasklet(receiver)(c)
       <stackless.tasklet object at 0x01BB8170>
       >>> sequence = range(4)
       >>> sequence
       [0, 1, 2, 3]
       >>> stackless.run()
       0
       1
       2
       3

.. method:: channel.__iter__()

   Channels can work as an iterator.  When they are used in this way, call
   overhead is removed on the receiving side, making it an efficient approach.
   
   Example - iterating over a channel::
   
       >>> def sender(channel):
       ...     for value in sequence:
       ...         channel.send(value)
       ...
       >>> def receiver(channel):
       ...     for value in channel:
       ...         print value
       ...
       >>> c = stackless.channel()
       >>> stackless.tasklet(sender)(c)
       <stackless.tasklet object at 0x01BB84F0>
       >>> stackless.tasklet(receiver)(c)
       <stackless.tasklet object at 0x01BB8170>
       >>> sequence = range(4)
       >>> sequence
       [0, 1, 2, 3]
       >>> stackless.run()
       0
       1
       2
       3       

.. method:: channel.next()

   Part of the :ref:`iteration protocol <typeiter>`.  Either returns the next value, or raises
   :exc:`StopIteration`.

.. method:: channel.open()

   Reopen a channel, see :meth:`close`.

   .. note::
   
      This functionality is rarely used in practice.

.. method:: channel.close()

   Prevents the channel queue from growing.  If the channel is not empty, the
   flag :attr:`closing` becomes ``True``.  If the channel is empty, the flag
   :attr:`closed` becomes ``True``.

   .. note::
   
      This functionality is rarely used in practice.

The following attributes can be used to select how the channel should behave
with regard to performed channel actions and the scheduling of involved
tasklets.

.. attribute:: channel.preference

   -1 prefer receiver (default).
   1 prefer sender.
   0 don't prefer anything.

.. attribute:: channel.schedule_all

   Setting this attribute to ``True`` overrides the value assigned to the
   :attr:`preference` attribute.  If set to ``True``, then any channel
   action will schedule to the next runnable.

Read-only attributes are provided for checking channel state and contents.

.. attribute:: channel.balance

   The number of tasklets waiting to send (>0) or receive (<0).
   
   Example - reawakening all blocked senders::
   
       >>> while channel.balance > 0:
       ...     channel.send(None)

.. attribute:: channel.closing

   The value of this attribute is ``True`` when :meth:`close` has been called.

.. attribute:: channel.closed

   The value of this attribute is ``True`` when :meth:`close` has been called
   and the channel is empty.

.. attribute:: channel.queue

   This value of this attribute is the first tasklet in the chain of tasklets
   that are blocked on the channel.  If the value is ``None``, then the
   channel is empty.
   
   Example - printing out the chain of tasklets blocked on the channel::
   
       >>> t = channel.queue
       >>> idx = 0
       >>> while t is not None:
       ...     print idx, id(t)
       ...     t = t.next
       ...     idx += 1
       ... else:
       ...     print "The channel is empty."

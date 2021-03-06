Why Mido?
==========

Messages as Objects
--------------------

Working with MIDI messages by manipulating the bytes is painful::

    NOTE_OFF = 0x80
    NOTE_ON = 0x90
    ...
    message = device.read()  # Returns [0x92, 0x40, 0x42]
    status_byte = message[0]
    if status_byte & 0xf0 in [NOTE_ON, NOTE_OFF]:
        is_note_on == status_byte & 0x10
        if is_note_on:
            message_type = 'note_on'
        else:
            message_type = 'note_off'
        channel = message[0] & 0x0f
        print('Got {} on channel {}'.format(message_type, channel))
        
This doesn't look much like Python! You could make some utility
functions on top of this to make it a little bit better, but it won't
get you far.

With Mido, you can instead do::

    message = port.receive()
    if message.type in ['note_on', 'note_off']:
        print('Got {} on channel {}'.format(message.type, message.channel))


Type and Value Checking
------------------------

Working directly with the bytes is also error prone. While MIDI data
bytes have a valid range of 0..127, the size of Python integers is only
limited by how much memory is available. If you make a mistake in your
computation of a data value, it could easily travel around undetected
until it blows up some seemingly unrelated part of your system. These
kinds of errors are tricky to track down.

Mido messages come with type and value checking built in. This happens
when you assign an out of range value to an attribute::

    >>> n = mido.Message('note_on')
    >>> n.channel = 2092389483249829834
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "./mido/messages.py", line 327, in __setattr__
        ret = check(value)
      File "./mido/messages.py", line 128, in check_channel
        raise ValueError('channel must be in range 0..15')
    ValueError: channel must be in range 0..15

and when you pass some nonsense as a keyword argument to the
constructor or the copy() method::

    >>> n.copy(note=['This', 'is', 'wrong'])
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "./mido/messages.py", line 316, in copy
        return Message(self.type, **args)
      File "./mido/messages.py", line 290, in __init__
        setattr(self, name, value)
      File "./mido/messages.py", line 327, in __setattr__
        ret = check(value)
      File "./mido/messages.py", line 181, in check_databyte
        raise TypeError('data byte must be an integer')
    TypeError: data byte must be an integer

This means that a Mido message object is always a valid MIDI message.

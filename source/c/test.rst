.. _c/test:

Firmware Doc
------------


Protocol
========

In order to communicate with LEIA, a simple communication protocol over uart is used. It allows to select a command, and execute it with specific data. The data produced by the command can finally be returned.

Only one data buffer :c:type:`command_cb_args_t` is used as the commands are executed one after the other in a sequential fashion. As there is no need to keep track of the result of the previous command, we can reuse the same buffer. The maximum size of our APDU payloads is 16384, and is mainly due to maximum buffers size in the MCU SRAM.

.. c:macro:: COMMANDS_BUFFER_MAX_LEN

        Constant to define the max size of the buffers used by the protocol (:c:type:`command_cb_args_t.buffer` and :c:type:`command_cb_args_t.response`).


.. c:struct:: command_cb_args

    Structure which handle the buffer send to a callback, and a buffer for the response of the callback.

        .. c:member:: uint32_t buffer_size

                Actual size of ``buffer``.

        .. c:member:: uint8_t buffer[COMMANDS_BUFFER_MAX_LEN]

                Buffer for the data which is send to the callback.

        .. c:member:: uint32_t response_size

                Actual size of the ``response`` buffer.

        .. c:member:: uint8_t response[COMMANDS_BUFFER_MAX_LEN]

                Buffer for the data produced by the callback.


To define a command, one have to create a :c:type:`command_t`. It needs a simple name (:c:type:`command_t.name`), which is used when debugging, a byte which is used to identify the command (:c:type:`command_t.o_command`), the maximal number of byte that can be send to the command (:c:type:`command_t.max_size`) and finally a callback to the function that will handle the command (:c:type:`command_t.callback`).

.. c:struct:: command_t

     Structure which describe a command to be exposed through LEIA API.

        .. c:member:: char name[30]

                Command name

        .. c:member:: uint8_t o_command

                Char which identify the command

        .. c:member:: uint32_t max_size

                Data max size (<= COMMANDS_BUFFER_MAX_LEN)

        .. c:member:: cb_command callback

                Callback to the function

.. c:struct:: protocol_config_pts_t

     Structure which codes the parameters to use when doing PTS negotiation.

        .. c:member:: uint8_t protocol

                Actual protocol to use:

                * 0 if no protocol is forced,
                * 1 for T=0,
                * 2 for T=1.

        .. c:member:: uint32_t etu

                ETU value.

        .. c:member:: uint32_t freq

                Actual frequency to use: 

                * 0 for the default one,
                * x for forcing a value.

        .. c:member:: uint8_t negotiate_pts

                * 0 for no negotiation
                * 1 for enabling negotiation           

        .. c:member:: uint8_t negotiate_baudrate

                * 0 for no baudrate negotiation
                * 1 for enabling baudrate negotiation

.. c:struct:: protocol_config_trigger_set_t
        
        Structure which code a strategy and an index to store the strategy at.

        .. c:member:: uint8_t index

                The index of the bank where the strategy will be saved.

        .. c:member:: trigger_strategy_t  strategy

                The strategy to save.

.. c:function:: uint8_t protocol_get_timers(SC_Card *card, command_cb_args_t *args)
        
        Callback to return timers values.

.. c:function:: uint8_t protocol_send_APDU(SC_Card *card, command_cb_args_t *args)

        Callback to process an APDU.

.. c:function:: uint8_t protocol_configure_pts(SC_Card *card, command_cb_args_t *args)

        Callback to configure a smartcard.

.. c:function:: uint8_t protocol_trigger_set_strategy(SC_Card *card, command_cb_args_t *args)

        Callback to set a trigger strategy.

.. c:function:: uint8_t protocol_trigger_get_strategy(SC_Card *card, command_cb_args_t *args)

        Callback to get a trigger strategy.

.. c:function:: uint8_t protocol_is_card_inserted(SC_Card *card, command_cb_args_t *args)

        Callback to check if the smartcard is inserted in LEIA.

.. c:function:: uint8_t protocol_reset_card(SC_Card *card, command_cb_args_t *args)

        Callback to reset the smartcard.

.. c:function:: uint8_t protocol_get_ATR(SC_Card *card, command_cb_args_t *args)

        Callback to send the ATR.

.. c:function:: int protocol_read_char_uart(volatile s_ring_t* ring_buffer, char* command)

        Read data from the uart, and put it in a ring buffer.

.. c:function:: void protocol_parse_cmd(volatile s_ring_t* ring_buffer)
        
        Parse a ring buffer to find a command to execute, call the corresponding callback.



Timers
======

It is possible to get the timing of APDU commands (the time taken by the smart card
to process the APDU command and send the response). We actually return two timers:
``delta_t`` and ``delta_t_answer``. ``delta_t`` is the global time take by the response
from the time we send the APDU until the last byte of the response. ``delta_t_answer`` is
the time taken between sending the APDU and the acknowledgement of the smart card (usually 
first byte of the response).

The Timers are usually sent back for every command of the protocol in the response, with 
zero values when timing such commands has no sense.


Triggers
========

On LEIA, there is a triggers handling mode where a dedicated PIN is put high and
then low on some events. Actually, two PINs are used for triggers: an internal trigger
dedicated to the LEIA board, and a ChipWhisperer trigger on the 20-pins connector
to allow compatibility with the CW ecosystem.

The events on which we can configure a trigger are mainly related to ISO7816-3 steps.
For instance, we can ask for a trigger just before the ATR (Answer To Reset) 
of the smart card with :c:type:`TRIG_GET_ATR_PRE`, just after this ATR reception with
:c:type:`TRIG_GET_ATR_POST`, etc. The list of events for which a trigger exists are the following: 

.. c:macro:: TRIG_GET_ATR_PRE

This trigger is placed at the beginning of the ATR reception.

.. c:macro:: TRIG_GET_ATR_POST

This trigger is placed at the end of the ATR reception.

.. c:macro:: TRIG_PRE_SEND_APDU_SHORT_T0

This trigger is placed at the beginning of T=0 short APDU send command.
 
.. c:macro:: TRIG_PRE_SEND_APDU_FRAGMENTED_T0

This trigger is placed at the beginning of T=0 fragmented (extended and case 4) APDU send
command.

.. c:macro:: TRIG_PRE_SEND_APDU_T1

This trigger is placed at the beginning of T=1 APDU send command.

.. c:macro:: TRIG_POST_RESP_T0

This trigger is placed at the end of T=0 response coming from the smart card.

.. c:macro:: TRIG_POST_RESP_T1                 

This trigger is placed at the end of T=1 response coming from the smart card.


.. c:macro:: TRIG_PRE_SEND_APDU

This trigger is placed at the beginning of APDU send command (either T=0 or T=1,
this is an abstraction for :c:type:`TRIG_PRE_SEND_APDU_SHORT_T0` or :c:type:`TRIG_PRE_SEND_APDU_FRAGMENTED_T0`
or :c:type:`TRIG_PRE_SEND_APDU_T1`.

.. c:macro:: TRIG_POST_RESP

This trigger is placed at the end of response coming from the smart card (either in
T=0 or T=1). This is an abstraction for :c:type:`TRIG_POST_RESP_T0` or :c:type:`TRIG_POST_RESP_T1`.

.. c:macro:: TRIG_IRQ_PUTC

This trigger is placed each time a byte is sent to the smart card on the ISO7816-3
half duplex line.

.. c:macro:: TRIG_IRQ_GETC

This trigger is placed each time a byte is received from the smart card on the ISO7816-3
half duplex line.



Configuring a trigger is quite simple in Python using the :c:type:`set_trigger_strategy` function:

.. code-block:: python
    import smartleia as sl

    reader = sl.LEIA('/dev/ttyACM1')

    reader.set_trigger_strategy(0, [sl.TriggerPoints.TRIG_GET_ATR_PRE, sl.TriggerPoints.TRIG_GET_ATR_POST, sl.TriggerPoints.TRIG_PRE_SEND_APDU_T1], delay=0, single=0)

This sets up the trigging strategy at index 0 with three events: :c:type:`TRIG_GET_ATR_PRE`, :c:type:`TRIG_GET_ATR_POST` and :c:type:`TRIG_PRE_SEND_APDU_T1`, with a
0 milliseconds delay and a permanent (not single) trigging style.

There are 4 possible strategies (index from 0 to 3) with a maximum of 10 events per strategy. The delay in milliseconds puts an offset in the
time the trigger happens.

At any time, it is possible to get the strategies states using :c:type:`get_trigger_strategy(index)`:

.. code-block:: python
    strat1 = reader.get_trigger_strategy(0)

    TriggerStrategy(single=0, delay=0, point_list=[<TriggerPoints.TRIG_GET_ATR_PRE: 1>, <TriggerPoints.TRIG_GET_ATR_POST: 2>, <TriggerPoints.TRIG_PRE_SEND_APDU_T1: 16>], point_list_trigged=[<TriggerPoints.TRIG_GET_ATR_PRE: 1>, <TriggerPoints.TRIG_GET_ATR_POST: 2>, <TriggerPoints.TRIG_PRE_SEND_APDU_T1: 16>], cnt_list_trigged=[1, 1, 3], event_time=[415121, 429329, 3038120])

The :c:type:`point_list_trigged` list gives the events that have been triggered, the :c:type:`point_list_trigged` list provides counters of how many times each event
has been triggered, and the :c:type:`event_time` shows an absolute time (using the reader internal clock) of when the last trigger of each event happened.

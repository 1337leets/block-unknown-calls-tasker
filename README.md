# Block Unknown Calls (Tasker)

A Tasker profile that cuts off calls from numbers that are not in my contacts, mostly spam and sales calls, but
still lets a real person through if they call again within five minutes. Callers from personal numbers get a short
SMS that explains this.

## How it works

The profile fires on an incoming call from a number that is not in the contacts (`!C:ANY`). Its task then

1. copies `%CNUM` into a local variable straight away, because `%CNUM` can change once the call ends
2. shows a notification with the number
3. looks the number up in two global arrays, `%RepeatNums` and `%RepeatTimes`, that hold each unknown caller and the
   time of their last call
4. if the same number called less than 300 seconds ago, shows a "repeat caller" notification and stops, so the call
   rings through normally
5. otherwise stores the number and the current time, replacing an older entry for the same number
6. turns airplane mode on, waits two seconds and turns it off again, which drops the call
7. stops here for corporate numbers starting with `0212`, `0216`, `0850` or `0800`
8. for any other number, waits five seconds and sends the SMS below

```
Hello, I can't answer the phone right now. This number is not in my contacts. You can leave a message,
or if it is urgent, call again within 5 minutes.
```

The prefixes are Turkish. `0212` and `0216` are Istanbul landlines, which are where most call centre calls come
from, and `0850` and `0800` are company and toll-free numbers. They get no SMS because nobody reads it there.

## Installation

1. Copy `Block_Unknown_Allow_Repeat.prf.xml` to the phone.
2. In Tasker, long-press the Profiles tab, choose Import Profile and pick the file.
3. Edit the Send SMS action if you want a different message, and the If condition if your corporate prefixes differ.

## Requirements

- Tasker
- Permission for Tasker to read the phone state and contacts and to send SMS
- `WRITE_SECURE_SETTINGS` for Tasker, so it can toggle airplane mode without root. It is granted once from a computer
  with ADB.

```bash
adb shell pm grant net.dinglisch.android.taskerm android.permission.WRITE_SECURE_SETTINGS
```

## Limitations

- The prefix check expects the number in national format (`0212...`). A number that arrives as `+90212...` is
  treated as personal and gets the SMS.
- Every blocked personal call sends an SMS, which may cost money depending on the plan.
- Airplane mode also cuts mobile data and any other call for those two seconds.
- The arrays grow by one entry per new unknown number and are never trimmed.

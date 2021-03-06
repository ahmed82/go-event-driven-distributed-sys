# NATS
```
kubectl apply -f nats.yaml
kubectl get pod |grep nats-box
```
```
Windows
 kubectl exec -it nats-box-9bbfb78bc-fpcvc -- //bin//sh -l
Linux/Mac 
 kubectl exec -it nats-box-9bbfb78bc-rcblq -- /bin/sh -l
```
## Let's Publish!
Think of publishing as broadcasting to a radio channel.
In this case the `test.x` radio channel.

The servers in my NATS cluster are named nats, nats2 and nats3.

```
nats -s nats pub 'test.x' x1
```

Now let's have a subscriber "tune-in".
```
nats -s nats2 sub 'test.x'
```

Why didn't the subscriber receive the message?
-- The subscriber tuned in late.


Now let's have another subscriber "tune-in" to ALL test channels.
```
nats -s nats3 sub 'test.>'
```

## NATS delivers messages  "at most once"
If a tree falls in a forest, and no one is around ...

If we publish on NATS and no one subscribes,
the message disappears into thin air.

---
# jetstream
Can NATS do better?

It certainly can! Just enable Jetstream within NATS itself.

```
docker run --rm -it nats:2.2.5 --jetstream
```
---------------------------------------------

The NATS cluster deployed to kubernetes already has jetstream enabled.

## Let's record our messages into a stream
Think of a stream as a log file. I will create a stream named `tst`.

First we need to feed that stream with a source of messages.
A NATS Stream has some interesting properties.
```
nats -s nats stream add


nats -s nats stream info tst

nats -s nats pub test.x x1
nats -s nats stream view tst

nats -s nats pub test.x.y xy2
nats -s nats req test.x x3
```

## We have logged a Stream. How do we use it?
I will create two consumers, one only interested in `test.x` .

And the other interested in all of `test`.

```
nats -s nats consumer add

nats -s nats2 consumer next tst x   --timeout=1h
nats -s nats3 consumer next tst all --timeout=1h
```

## Other stream functions

```
nats -s nats str report
nats -s nats s view tst 3
nats -s nats s rmm tst N

nats -s nats consumer list tst
nats -s nats c copy tst all all1
nats -s nats3 consumer next tst all1 --timeout=1h

nats -s nats s purge tst
nats -s nats s rm tst

```

## Clean up
```
kubectl delete -f base/nats.yaml
```

## Presentation and code download

.link https://github.com/siuyin/present-intro-nats

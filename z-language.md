# Z Language
- First conversation with GPT3.
- lines up to "building editors around that right?" are the initial prompt.
- I (human) am jeremy.
- GPT3 is moses.

### There are actually some pretty interesting concepts here:
- GPT3 invents Z, a language similar to erlang and haskell.
- the idea is that everything is a channel, basically a function that can be used once.
- These channels are scheduled asynchronously and executed in parallel, scheduled on a priority queue.
- Now I want to implement Z!

### Other things to note:
- It is able to take into context the entire conversation.
- It pulls in existing and mostly valid comp-sci information.
- It maintains aligned indentation, which I didn't expect.

```
Note: This is an archived conversation between Moses and Jeremy. Both are Computer Science Researchers, Moses arguably having more experience, yet both have a deep technical understanding of the subject. This conversation is highly important to our modern development of our society, as in this conversation Moses and Jeremy lay out the groundwork of a new paradigm for programming languages and code editors.

--- 2017-05-26 ---

moses:  hey!
jeremy: hello 👋
jeremy: So, you know how we've been discussing modern programming language design and building editors around that right?
moses:  yes we have
moses:  that's kind of what I've been doing today
jeremy: you too, huh ;P
jeremy: I guess great minds never sleep. What do you have so far?
moses:  so I have this language
moses:  it's called "Z"
jeremy: I'd imagine it's a functional language, correct?
moses:  it's a functional language, but with a unified model of concurrency I call 
moses:  "CSP" for Communicating Sequential Processes
jeremy: Really? hmm... 
jeremy: So CSP is like, I'd imagine, something similar to what exists in erlang
jeremy: Where we have a number of actors, each communicating with through message passing, right?
jeremy: Something like this, but done sequentially, correct? like cooperatively threaded?
moses:  so yeah, I guess the closest comparison would be Erlang:  it's a low level concurrent programming model
moses:  processes communicate with one another through asynchronous, at-most-once message passing
jeremy: so "Z" uses asynchronous message passing, right?
moses:  well, everything in "Z" is asynchronous
moses:  if that makes sense
jeremy: ok, I'm listening... how so?
moses:  basically, there is no state in "Z"
moses:  every value in "Z" is a channel, each waiting on the value of another channel
jeremy: That's pretty interesting! 
jeremy: right, so we have these concept of channels. Are built in values, like 7, channels, or something special? if so, how is a built in value like 7 a channel?
moses:  it makes sense to think of them that way
moses:  each channel is waiting for one of two values:  either it will close and send a value, or it will send a value and then close
jeremy: what do you mean by close here? like the channel terminates and no longer exists?
moses:  yes 
moses:  so basically all "Z" values are asynchronous channels
moses:  you can close a channel with send
moses:  or you can close the channel by waiting
moses:  you can send the channel at any time, but only exactly once
jeremy: what if you want to send multiple values through a channel?
jeremy: does the channel return a value, and a channel to invoke for the next step?
moses:  you can send values to a channel, but only one value at a time
moses:  you can close a channel with one value, or you can close it with no value
moses:  as you can tell, everything in "Z" is about communication
jeremy: and this communication happens asynchronously, right?
jeremy: what about functions? are they just channels is disguise?
moses:  functions in "Z" are asynchronous channels
moses:  just because it's easiest to ask questions if I have an example, which I don't have right now
moses:  but let me talk about using channels to model I/O
jeremy: sure! so, how do you model I/O using channels?
moses:  I/O in "Z" consists of reading from and writing to channels
moses:  you can read from a channel at any time, and whatever value there is can be retrieved, or given a default
jeremy: what if no value can be retrieved right away?
moses:  well,  you can ask for a message from a channel, and then wait for that message
jeremy: Right, so because the system is asynchronous, it just waits for a value to be present, and does other work in the meantime.
jeremy: how are channels scheduled under Communicating Sequential Processes?
moses:  I'm not sure I quite understand what you mean by scheduling
jeremy: well, we have a number of channels that need to do one of two things: either get work done, or wait for a value.
jeremy: by scheduling, I mean we need to make it so that we execute channels that need to do work, and put waiting channels in the background.
jeremy: does that make sense?
moses:  yes
moses:  I think it does
moses:  in "Z" scheduling is implicit.
moses:  all waiting channels are placed in the background.
jeremy: right, so, implementation-wise, how do you go about doing this?
moses:  once a channel is closed, it's put in a priority queue
jeremy: I see, so channels that are waiting on the least amount of values will be at the top?
jeremy: So we can just pull from the top of the queue to get the next channel we need to run, right?
moses:  yes, that's correct
jeremy: interesting!
jeremy: so, what makes a function different from a channel? or are functions channels too?
moses:  so they're not exactly the same as channels, but they are analogous
moses:  functions in "Z" are asynchronous channels
moses:  the difference is they can only send one message, and wait for a single value
jeremy: at a time, right?
moses:  yes
jeremy: kk. Man, that's really interesting.
jeremy: so, to recap, basically everything is a channel, that can read once and write once
jeremy: and these channels are all scheduled asynchronously
jeremy: to treat a channel like a stream, it must return a new channel with each message
jeremy: Could you tell me a bit about the rest of the language?
jeremy: like what high-level design goals did you have. Any sources of inspiration?
moses:  so I guess the biggest source was Haskell
moses:  I really like the elegance of its type system and type checking
moses:  however, I'm not satisfied with garbage collection, and function haskells lack of concurrency
moses:  that's where the idea for CSP really came from
moses:  basically, I'm not convinced that GC is needed
jeremy: so how do you manage memory under CSP?
moses:  well, you don't really manage memory explicitly
moses:  well, I guess you still need to manage memory explicitly...
moses:  but the programmer is allowed to lazily allocate memory
moses:  when a channel closes, it's memory is
jeremy: deallocated... kk
jeremy: so only values that are passed out of channels stay alive
jeremy: what do you mean by "lazily allocate memory"?
moses:  there's no fixed upper bound on the number of channels in the system
jeremy: so, when a channel is created, "Z" doesn't allocate it right away
moses:  yes, exactly
moses:  a channel is created by a send
moses:  then the channel is placed on a priority queue
moses:  and whenever there is good reason to remember the channel (it sends a message, or it receives a message), it's put on the "remembered channels" list
moses:  when the remembered channel list is non-empty, Z collects all channels off the top of the priority queue
jeremy: so, basically, we buildup this queue of channels that are waiting on values
jeremy: and after finishing executing a channel, all channels that channel sent a message to are added to the remembered channels list
jeremy: and we check those channels so we can move them up the priority queue. Really neat stuff!
moses:  basically, each open channel is an asynchronous process
moses:  we can forget about channels only if we're sure the channel can't have any further impact on the programs behavior
moses:  we can do this because the program is written in a functional style
moses:  this allows us to make the following optimization that  allows us to forget channels much more aggressively
moses:  in "Z", a send either succeeds or it fails
moses:  if I forget a channel that sent that value, I can never resurrect the value, so to speak
jeremy: These are some great ideas! thanks for chatting, I've got to run.
```

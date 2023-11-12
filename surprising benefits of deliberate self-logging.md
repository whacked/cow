The surprising benefits of disciplined, deliberate self-logging
===

In the summer of 2018, I mysteriously started developing acid reflux symptoms, which spurred me to take disciplined records of my diet. To help with the increased logging load, I ended up writing an [app](./biologger.md), which supplanted all of my earlier attempts of logging in plain text or other half-baked solution like fitbit [^fitbit]. Aside of turning me into _that colleague_ who logs everything he eats, there was a subtle yet life-changing effect: perceptual sensitivity.

[^fitbit]: fitbit's export data structure is a total mess:
    - food and water intake granularity is at the day level, which is silly
    - the date format alters between `MM/DD/YY` and `YYYY-mm-dd`
    - there are `calories` in the root of `loogedFood` as well as `NutritionalValues`
    - the units are `FL_OZ` in water (through `measurementUnit`) and `oz` in food (through `unit.name`, which is obviously a separate table, _different_ from that used for water)
    - why does resting heart rate have a low fidelity `dateTime` like `09/24/17 00:00:00`, but include the same `date` like `09/24/17` value in `.value.date`?
    - time data has [no timezone information](https://community.fitbit.com/t5/Web-API-Development/Clarification-on-time-zones-and-date-time-values/td-p/925255) and is recorded in string

Let me explain.

The act of logging a personal, internal state is an act of observation. You direct your mental attention at an internal phenomenon to discern what it is. For example, here are some actual entries from my logs of "food coma" responses:
```
{"time": "2013-08-16T14:07:11-04:00", "entry": "recovered from coma around 5 minutes ago; coma lasted for probably 15 minutes"}
{"time": "2015-02-15 16:31:11-05:00", "entry": "wake from coma, ~5min. severe, total LoC"}
{"time": "2015-03-19T16:37:37-04:00", "entry": "recover from minor coma. maybe < 5 min; rapid low energy"}
{"time": "2016-07-04T13:40:55-07:00", "entry": "downcycle 4/10"}
{"time": "2016-06-13T17:00:00-07:00", "entry": "les 7/10 with intrusions"}
{"time": "2016-07-17T18:57:04-07:00", "entry": "frequent intrusions 6/10"}
{"time": "2020-05-16 21:04:36-0700", "entry": "brain slip", "category": "mental state"}
```

Over time, I create new vocabulary to describe details that I notice at the time of capture. Some of these vocabularies get superseded by others (what I called `intrusion` is in fact a microsleep). This kind of evolution may happen across months. It makes for a nasty set of data for quantitative analysis later, but at the same time, the observer is picking up more and more details about the _surrounding phenomena_. The `downcycle` is actually a response that precedes falling asleep.

Like bird watching, poetry reading, or microscope viewing, observing the internal state is a learning process, where repeated exposure allows the brain to practice and develop expertise in finding details in what it's observing. In my case, I was able to glean a surprising amount of information after focused observation in this way. I have a finer perception of the "food coma" response than during my teens, when it was simply a "feel sleepy, fall asleep, wake up" matter.

I've written about the phenomenon of increasing tactile acuity in my investigation of [learning braille]; this is similar in nature. Another example where I've noticed increased perceptual "resolution" is from observing... ~~poo~~ stool. Well, that's a separate topic.

The takeaway: through systematic and conscious repetition in observing a single target, the brain fine-tunes its perception, uncovering new features that _become obvious_. Yet, to the untrained brain, they easily remain _hidden from awareness_.

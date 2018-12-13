# How things should works

```js
let event = await Event.findById(req.params.id)
                       .include('comments', attr: '_id summary')
                       .where('comments.summary': "dude")
// event == { title: " ", comments: [ { _id: "asfsdafasdffasd", summary: "dude" } ] }
```

* What about include "where"

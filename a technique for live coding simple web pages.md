Say you want to quickly show some custom information in a nice tabular format, and decide that an HTML/SVG page would do the job. Since there are a lot of rows, you write a script to generate them instead of writing lots of tags. After wrapping up the output in some extra HTML, now you have a nice page you can view in the browser.

Now let's say you want to iterate on the formatting and styling of the page. You could tweak the code, maybe rerun your script, and refresh. To speed that up, you could use a live reload browser extension. To automate the browser reloading whenever you make changes, you could bring in a framework like [next.js](https://nextjs.org/).

I wanted these conveniences, but something lighter. Here is a handy method I've been using in the recent months.

There are 2 core components:
1. a container HTML page that connects to a websocket
2. [websocat](https://github.com/vi/websocat) for data relay (this program is _ridiculously_ useful)

After that, you just send the HTML you want to display (a file, script, curl output, whatever) to websocat. For refresh-on-save, you can use [watchexec](https://github.com/watchexec/watchexec) or a file monitor of your choice, like [entr](https://github.com/eradman/entr), or plain looping in a script.

## open the container page in a browser

this example contains the bare minimum logic of connecting to a websocket and updating a DOM container.

```html
<!-- display.html -->
<html>
<body>
    <div id="container"></div>
    <script>
        $container = document.getElementById('container')
        const connectLoop = () => {
            const connection = new WebSocket(`ws://localhost:9999`)
            connection.onopen = (event) => {
                console.info('socket connected')
            }
            connection.onclose = () => {
                console.warn(`socket closed; retrying`)
                setTimeout(connectLoop, 5000)
            }
            connection.onmessage = function (event) {
                let message = (event.data ?? '').trim()
                let newElement = document.createElement('div')
                newElement.innerHTML = message
                $container.innerHTML = ''
                $container.appendChild(newElement)
            };
        }
        connectLoop()
    </script>
</body>
</html>
```

## start websocat

```sh
websocat -t ws-l:127.0.0.1:9999 broadcast:mirror: --exit-on-eof
```

once connected, whatever you send to port 9999 will get displayed in the browser

## push HTML to the browser

if your HTML is from a file on disk, you can simply pipe it into websocat:

```sh
cat your-file.html | websocat ws://localhost:9999
```

the [bootleg](https://github.com/retrogradeorbit/bootleg) library allows us to use `hiccup` with [babashka](https://github.com/babashka/babashka) to push frames as an animation:

```clojure
;; mypage.bb.clj
(require '[babashka.pods :as pods])
(pods/load-pod 'retrogradeorbit/bootleg "0.1.9")
(require '[pod.retrogradeorbit.bootleg.utils :as utils])
(require '[babashka.process :as p :refer [process]])

(defrecord Circle
    [radius
     color
     step-size
     x x-velocity
     y y-velocity
     ])

(defn send-output! [payload number]
  (if nil  ;; write files instead?
    (spit (format "frame-%03d.svg" number) payload)
    (-> (process ['echo payload])
        (process '[websocat "ws://localhost:9999"]))))

(let [$steps 333
      $width 900
      $height 700
      $circles (concat
                [(->Circle 50 "pink" 20 (* $width 0.1) 1.2 (* $height 0.2) 1.0)
                 (->Circle 30 "olive" 10 (* $width 0.7) 0.8 (* $height 0.6) 1.3)
                 (->Circle 20 "skyblue" 10 (* $width 0.4) 1.4 (* $height 0.3) 1.2)]
                (->> (range 99)
                     (map (fn [_]
                            (->Circle 15 "#cccccc" 15 (/ $width 20) (rand) (* (/ $height 20) 19) (rand))))))
      out-of-bounds? (fn [min-position max-position velocity position radius]
                       (or (and (< velocity 0) (< position (+ min-position radius)))
                           (and (> velocity 0) (< (- max-position radius) position))))]
  (loop [remain-steps (range $steps)
         circles $circles]
    (when-let [step (first remain-steps)]
      (-> [:svg
           {:xmlns "http://www.w3.org/2000/svg"
            :xmlns:xlink "http://www.w3.org/1999/xlink"
            :width $width
            :height $height}
           [:g
            (->> circles
                 (map (fn [{:keys [x y radius color]}]
                        [:circle
                         {:cx x
                          :cy y
                          :r radius
                          :fill-opacity 0.5
                          :fill color}])))]]
          (utils/convert-to :html)
          (send-output! step))
      (recur (rest remain-steps)
             ;; next step's circles
             (->> circles
                  (map (fn [{:keys [x y step-size radius x-velocity y-velocity] :as circle}]
                         (let [next-x (+ x (* step-size x-velocity))
                               next-y (+ y (* step-size y-velocity))]
                           (assoc circle
                                  :x next-x
                                  :x-velocity (if (out-of-bounds? 0 $width x-velocity next-x radius)
                                                (- x-velocity)
                                                x-velocity)
                                  :y next-y
                                  :y-velocity (if (out-of-bounds? 0 $height y-velocity next-y radius)
                                                (- y-velocity)
                                                y-velocity))))))))))
```

![](./img/2022/10/2022-10-26_babashka-animation.svg)

the combined output above was generated using [svgasm](https://github.com/tomkwok/svgasm):

```sh
svgasm -d 0.02 -i 99 frame-*.svg -o animated.svg 
```

## start watchexec and code away

```sh
watchexec -r -w . -- bb mypage.bb.clj
```

This isn't quite like the REPL-driven live coding that is enabled by [figwheel](https://figwheel.org/), but for small pages, the push-on-save feels instantaneous, making for a tight code-to-display feedback loop.

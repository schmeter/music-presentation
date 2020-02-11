# Music

This is about how to put your own music into an app that's running in the browser, so that it can be provided to the rest of the world in a nice and accessible way without you worrying about legal issues related to platforms where would normally upload your files.  


## Past

To provide music to visitors of your website was already possible with an early version of Internet Explorer, where you could put a file as a background noise into your HTML code, like to be seen here:  
https://developer.mozilla.org/en-US/docs/Web/HTML/Element/bgsound  

This was quite annoying and uncomfortably to use, so that visitors mainly would not return to that site again.  


## Present

Today there' an easy to use and comfortable to handle way to provide audio files with HTML5 Audio:  
https://developer.mozilla.org/en-US/docs/Web/HTML/Element/audio  

At least you only have to define a source file and the existence of controls to have a working audio player in the browser.  

So why would you need more than that?  


## Objectives to reach

You already created a web-based music player in the browser, but:  

You want to display playlists to choose audio files from.  
You want to have an analyser to display animations related to the current playing audio.  
You want to display lyrics or information regarding current audio.  
You want to make your player look like something you really adore.  


## Work to do

### A few questions

What do I really want to do?  

Build an own audio interface?  
- Nope. Use native elements.  
- Use styling on that or rely on standards.  
- HTML5 Audio: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/audio  

How to get audio content analysed?  
- Use Web Audio API: https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API  

How to display the analysis?  
- Use canvas: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API  

How to handle app state?  
- Use React.js component state and Redux only when needed.  
- Local storage for user settings.  

Device support?  
- Default Android and iOS browsers.  
- Chrome, Firefox and Safari on Desktop.  

What else?  
- Use a sitemap.  
- Have update cycles.  


### Toolchain setup

Technologies to be used: React, Redux, Sass.  

Create base configuration:  
- .nvmrc  
- .editorconfig  

Prepare linting:  
- ESLint  
- Sass Lint  

Use tests:  
- Jest  


## Solution

### Task: Data collection

To display a music library, I need to read the data available on disk and make it available inside the app.  

- Scan folders with mp3 files, images, lyrics files.  
- Read ID3 tags, add cover images, parse markdown files.  
- Configure order and display mode of artists and albums.  
- Provide visible and hidden content for different users.  
- Store data in audio model.  

Tools:  
- grunt: run the tasks  
- grunt-tree: build file trees  
- id3-parser: read ID3 tags  
- markdown: parse markdown content  

---


### Task: Player interface

To have a player for the files in the browser, I need to provide an interface.   

Audio tag:
- Use 1 audio element.  
- Provide native controls.  
- Use mp3 support only, since there's only mp3 files.  
- Switch sources by user interaction.  
- Handle events.  

```
    <audio
        className="audio"
        controls
        ref={this.audio}
        onPlay={this.handlePlay}
        onPause={this.handlePause}
        onEnded={this.handleEnded}
        onError={this.handleError}
    />
```
---


### Task: Positioning

To have the player available at all times, it has to be present. 
There's a dynamic browser interface in mobile browsers which makes the player element move up and down.  
I want continuous availability in mobile browsers in the same place at all times.  

- Fix screen to prevent scrolling issues.  
- Restrict body size.  

```
    body {
        width: 100%;
        height: 100%;
        overflow: hidden;
    }
```

- Use scroll container.  
- Add native scroll feeling in iOS.  

```
    main {
        position: absolute;
        top: 0;
        right: 0;
        bottom: 0;
        left: 0;
        overflow: auto;
        overflow-y: auto;
        -webkit-overflow-scrolling: touch;
    }
```

- Position the interface.  

```
    audio {
        position: fixed;
        right: 0;
        bottom: 0;
        left: 0;
    }
```

---


### Task: Styling

Safari and Firefox look fine, there's no need to change except removal of rounded corners.  
Only minor styling available on DOM.  

- Apply basic styles

```
    audio {
        width: 100%;
        height: 30px;
        background: radial-gradient(#fff4e1, #505050);
    }
```

Then there's that ugly thing in Chrome.  

- Activate Shadow DOM.  
- Apply styles to shadow DOM elements (first approach).  

```
    &::-webkit-media-controls,
    &::-webkit-media-controls-panel,
    &::-webkit-media-controls-enclosure,
    &::-webkit-media-controls-overlay-enclosure {
        visibility: hidden;
    }

    &::-webkit-media-controls-overflow-button,
    &::-webkit-media-controls-mute-button,
    &::-webkit-media-controls-play-button,
    &::-webkit-media-controls-timeline-container,
    &::-webkit-media-controls-current-time-display,
    &::-webkit-media-controls-time-remaining-display,
    &::-webkit-media-controls-timeline,
    &::-webkit-media-controls-volume-slider-container,
    &::-webkit-media-controls-volume-slider,
    &::-webkit-media-controls-seek-back-button,
    &::-webkit-media-controls-seek-forward-button,
    &::-webkit-media-controls-fullscreen-button,
    &::-webkit-media-controls-rewind-button,
    &::-webkit-media-controls-return-to-realtime-button,
    &::-webkit-media-controls-toggle-closed-captions-button {
        visibility: initial;
    }
```

Advantage: Styles applied.  
Disadvantage: Some controls are lost.  

But: Suddenly Chrome changes things about HTML5 Audio again.  

- Apply styles to shadow DOM elements (second approach).  

```
    &::-webkit-media-controls,
    &::-webkit-media-controls-panel,
    &::-webkit-media-controls-enclosure,
    &::-webkit-media-controls-overlay-enclosure {
        background-color: #fff4e1;
    }
```

Advantage: Controls are visible.  
Disadvantage: Still have to rely on Chrome's native output.  

---


### Task: Make the app accessible and play the audio

I use React.js components to display the data collected and provide links to audio files which should then be loaded into the audio player.  

- Display the item and handle the click event.  

```
    <File
        path={file.path}
        onClickFile={this.handleClickFile}
    >
        {file.tag.title}
    </File>
```

- Hold the state.  

```
    audio: {
        library: {
            tracks: [],
            albums: [],
            artists: [],
        },
        activeIndex: -1,
        isPlaying: false,
        playToggle: false,
    }
```

- Tell the app about change.  

```
    const mapDispatchToProps = (dispatch) => ({
        setActiveIndex: activeIndex => dispatch(setActiveIndexAction(activeIndex)),
        togglePlay: () => dispatch(togglePlayAction()),
    });
```

- Display audio information from store.  

```
    <Image
        src={activeTrack.album.imgPath}
        alt={activeTrack.album.title}
    />
    <div
        className="lyrics"
        dangerouslySetInnerHTML={{ __html: activeTrack.lyrics }}
    />
```

- Connect audio events.  

```
    handlePlay() {
        const { setIsPlaying } = this.props;

        setIsPlaying(true);
    }

    handlePause() {
        const { setIsPlaying } = this.props;

        setIsPlaying(false);
    }

    handleError() {
        const { setIsPlaying } = this.props;

        setIsPlaying(false);
    }

    handleEnded() {
        const { nextIndex, setActiveIndex } = this.props;

        setActiveIndex(nextIndex);
    }
```

- Add handling convenience with cursor navigation.  

```
    captureKeys(e) {
        switch (e.keyCode) {
            case 32:
                e.preventDefault();
                this.togglePlay();
                break;
            case 37:
                e.preventDefault();
                this.decreaseTimeFrame();
                break;
            case 39:
                e.preventDefault();
                this.increaseTimeFrame();
                break;
            default:
        }
    }
```

---


### Task: Analyse the audio

I need to use the Audio Context for grabbing the data from the audio file.  

- Create Audio Context.  

```
    analyse() {
        const { audio, isPlaying } = this.props;
        const canvas = this.canvas.current;
        const AudioContext = window.AudioContext;

        if (
            canvas
            && audio
            && AudioContext
            && !this.audioContext
            && isPlaying
        ) {
            this.audioContext = new AudioContext();
            const analyser = this.audioContext.createAnalyser();
            const source = this.audioContext.createMediaElementSource(audio);

            source.connect(analyser);
            analyser.connect(this.audioContext.destination);
            this.drawAnalyser(canvas, analyser);
        }
    }
```

In Safari: AudioContext or WebkitAudioContext?  
Not working for mobile Safari, but desktop.  
So: No analyser for iOS.  

But when to start the Audio Context?  
Chrome wants it on user interaction.  

- Check for user interaction.  

```
        componentDidMount() {
            this.analyse();
        }

        componentDidUpdate() {
            this.analyse();
        }
```

If you host your files "somewhere", you might not have access because of CORS: https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS.  
This will result in silence.  

- So activate headers on server side.  

```
    Header add Access-Control-Allow-Origin "*"
```

Now I want to display the analysed data.  

- Set up a canvas.  

```
    <canvas
        className="canvas"
        ref={this.canvas}
    />
```

- Style the canvas.  

```
    canvas {
        position: fixed;
        top: 0;
        right: 0;
        bottom: 0;
        left: 0;
    }
```

- Analyse the data: What's the content?  
- Provide waveform or frequency analysis.  
- Draw the data to the canvas.  

```
    drawAnalyser(canvas, analyser) {
        const { mode = 'frequency' } = this.props;
        const bufferLength = analyser.frequencyBinCount;
        const dataArray = new Uint8Array(bufferLength);

        canvas.width = bufferLength;
        canvas.height = 256;
        const ctx = canvas.getContext('2d');
        const barWidth = 1;
        const barDistance = 1.375;
        const paintCanvas = () => {
            if (mode === 'waveform') {
                analyser.getByteTimeDomainData(dataArray);
            } else {
                analyser.getByteFrequencyData(dataArray);
            }
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            dataArray.forEach((item, index) => {
                const barHeight = item * 3 / 4;

                if (mode === 'waveform') {
                    ctx.fillStyle = `rgba(255, 170, 0, 1)`;
                    ctx.fillRect(index, canvas.height - barHeight, barWidth, 1);
                } else {
                    ctx.fillStyle = `rgba(255, ${255 - item}, 0, ${1 - (item / canvas.height)})`;
                    ctx.fillRect(index * barDistance, canvas.height, barWidth, 0 - barHeight);
                }
            });
            window.requestAnimationFrame(paintCanvas);
        };

        paintCanvas();
    }
```

But: Some devices are simply too slow.  

- Make things configurable.  

```
    isAnalyserAllowed() {
        const { audio } = this.props;

        return audio && !isTouch() && configApp.analyser.active;
    }
```

There's lots more possibilities with Audio context, like adding equalizers, filters and so on.  

---


### More features

I want to provide a sitemap.txt.  

- Add the static routes AND the dynamic audio configuration.  

```
    https://example.com/
    https://example.com/audio/demoartist
    https://example.com/audio/demoartist/thealbum
    https://example.com/settings
```

I want to have update cycles.  

- Provide version data in index.html.  

```
    <script>
        var app = {
            rev: '45b3baa',
            time: '1554968510870'
        };
    </script>
```

- Generate a version.json containing the same data.  

```
    {
        "rev": "45b3baa",
        "time": 1554968510870
    }
```

- Read fresh JSON file on app start and check for updates, if page was cached.
- Provide a simple interface.  


```
    fetchJSON(`/version.json?${new Date().getTime()}`, true).then(data => {
        if (process.env.NODE_ENV === 'production') {
            const searchTime = parseInt(window.location.search.substr(1), 10);

            if (searchTime !== data.time && app.time < data.time) {
                if (window.confirm(i18n('app_update_available'))) {
                    window.location.href = `/?${data.time}`;
                }
            }
        }
    });
```

I want to persist user settings.  

- Use local storage.  
- Login state: Show hidden data.  
- Current audio file: Store user selection.  


## Demo

https://antipeter.de/


## Issues

- Autoplay next file on mobile is not possible, if browser tab is inactive.  
- Also on desktop, if JavaScript is not allowed in inactive tabs.  
- Audio image display on lockscreen is not possible.  
- Only partial lockscreen navigation.  


## Next steps / possibilities

- Add zip download.  
- Have server side rendering.  
- Work on SEO optimization.  
- Provide content sharing.  
- Build a progressive webapp.  
- Add a (React) native app with webframe.  
- Build an Electron app.  
- Integrate with Heroku / Netlify for continuous deployment.  


## Conclusion

Everything works (with constraints).  
There's styling possibilities (in Chrome).  
Audio context handling is not simple.  

Contributions are welcome:  
https://github.com/schmeter/music  

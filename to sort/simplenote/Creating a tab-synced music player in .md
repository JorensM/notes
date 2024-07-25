# Creating a tab-synced music player in React

**Warning, kind of a long read, here is the [source code](https://github.com/Printy-Studios/react-music-player) if you don't feel like reading the entire article**

Hi dear readers!

In a recent project, I was tasked with creating a music player. The requirements were as follows:

 * The player had to work and sync even with multiple tabs open
 * The player state needed not to be reset upon a refresh

It was certainly an interesting task and the features listed above were something I hadn't worked with before.

In this article I'd like to recreate this music player, and show you, step-by-step, how to do it.

Our player will have the following features:

 * Tab syncing
 * State persistance between refreshes
 * Play/Pause/Prev/Next buttons
 * Song info such as cover art and title/author
 * Songs will be fetched from the app's `public` folder, and you can easily add more songs by adding a sound file, cover art and a metadata file to the app's `public` folder

The project will be written in TypeScript, but you can use JS just as well (just omit any TypeScript syntax).

## Set up

I will assume that you already know how to set up a project. I will personally be using [Vite](https://vitejs.dev/) to set up my project.

After having set up our project, we will need to install 2 libraries (that I wrote myself) - [react-tab-state](https://www.npmjs.com/package/@printy/react-tab-state) and [react-persist-state](https://www.npmjs.com/package/@printy/react-persist-state). These libraries will allow us to sync our player instances between tabs and also persist them even after the page has been closed. If you want to learn how these libraries work (they're actually quite simple), I wrote 2 articles about that, which you can find by following the links below:

 * [How to keep state between page refreshes in React](https://dev.to/jorensm/how-to-keep-state-between-page-refreshes-in-react-3801)
 * [How to sync React state across tabs with workers](https://dev.to/jorensm/how-to-sync-react-state-across-tabs-with-workers-2mpg)

For icons, I will be using the following ones:

* [Play icon](https://www.svgrepo.com/svg/532511/play)
* [Pause icon](https://www.svgrepo.com/svg/522621/pause)
* [Next icon](https://www.svgrepo.com/svg/521767/next)
* [Previous icon](https://www.svgrepo.com/svg/521791/prev)

But you can of course use any icons you want.

## Project Structure

After having set up our project, let's walk through our project structure and which files we'll be writing

The project structure will be as follows:

 * `App.tsx` - The page where our music player will be displayed
 * `style.css` - Where our styles will reside
 * `useMusicPlayer.ts` - A hook for manipulating audio
 * `MusicPlayer.ts` - The music player component
 * `SongMetadata.ts` - Type for our song metadata objects
 * `util.ts` - Utility functions
 * `IconButton.tsx` - Reusable icon button with animations
 * `PlayButton.tsx` - Component for our play button
 * `TimeDisplay.tsx` - Component for displaying time in the playere player

Additionally, our `public` folder will have the following structure:

![](public-folder.png)

 * `cover_art/` - This is where we will keep the cover art for the songs. The file name should be of `.jpg` format and should match the ID of the song defined in `songs_metadata.json`
 * `icons/` - Folder with our icon files
 * `songs/` - Folder with our songs. The files should be of `.wav` filetype and their name should match the ID of the song
 * `worker.js` - Our worker file that will manage the information about the main tab (needed for syncing the players across tabs)
 * `songs_metadata.json` - This is where information about our songs will be, such as song ID, title and author.

Alright, now we know what our project structure will look like, we can start coding! If at any point you feel lost as to where a certain piece of code should go, feel free to refer to the [source code](https://github.com/Printy-Studios/react-music-player)!

## `useMusicPlayer.ts`

First of all, let's write our `useMusicPlayer` hook.

Let's begin by scaffolding our function

```
export default function useMusicPlayer() {

}
```

Next up, let's add all the necessary state

```
export default function useMusicPlayer() {
    const audio = useRef(document.createElement('audio'));

    const [isPlaying, setIsPlaying] = useTabState(false, 'is_playing');
    const [src, setSrc] = useState<string>("");
    //setCurrentTime is for local use only, for setting time use updateTime
    const [currentTime, setCurrentTime] = useTabState<number>(0, 'current_time');
    const [updatedTime, updateTime] = useState(0);

    const [isMainTab, setIsMainTab] = useState<boolean>(false);

    const [storedTime, setStoredTime] = usePersistState(0, 'stored_time');
    
    const [maxTime, setMaxTime] = useState(0);
}
```

Let's go through each line and see what kind of state they define

First we create an `audio` ref, this will hold a reference to our HTML audio element that we will use to play audio. We won't actually be showing this element on the page, we'll just be using it for playback.

Next we have `isPlaying`. This determines whether our player is currently playing or paused. You may notice that we're not using a regular `useState()` function to create this state. `useTabState()` is a special hook that I created that allows you to sync the given state across tabs. So if the value changes in one tab, it will also change in any other tabs for this page. The first argument is the initial value, and the second argument is a unique ID to identify our state.

Then we have a `src` state that determines the URL of the source sound file that the audio player should be playing.

Next we have `currentTime` and `updatedTime`. `currentTime` tells us the current time of our audio playback, and `updatedTime` is needed to act as a proxy when setting current time, so we don't run into an infinite loop when setting current time with our slider and syncing state across tabs. You may notice that `currentTime` also uses `useTabState`, this is because we want to sync the current time across tabs.

Next up is `isMainTab`. This tells us whether the current tab is the 'main' tab. Later we will be writing a small worker script that will determine which tab should be the main tab. We need this 'main tab' feature because we will want to have audio playing only from one tab if there are multiple tabs open.

`storedTime` will hold state for our 'stored' current time that will be saved into localStorage and persisted across refreshes. This state uses another one of my custom hooks - `usePersistState`, which creates a state that gets persisted across page refreshes. Just like with `useTabState`, the first arg is the initial value and the second arg is the localStorage key to store the state under.

And finally `maxTime` tells us the max time (length) of our current song.

Alright, now that we have set up our state, let's write our `useEffect`s

First off let's write the `useEffect` that gets called upon mounting of our app. Here we will set up our connection with the worke and add event listeners to the `audio` element:

```
useEffect(() => {
    worker.port.start();

    setInterval(() => {
        worker.port.postMessage({
            type: "ping"
        });
    }, 1000)

    worker.port.onmessage = (e) => {
        const data = e.data;
        switch(data.type) {
            case 'set_main_port': {
                setIsMainTab(true);
                break;
            }
            case 'unset_main_port': {
                setIsMainTab(false);
                break;
            }
        }
    }

    audio.current.preload = 'metadata';
    
    audio.current.ondurationchange = () => {
        setMaxTime(audio.current.duration)
    }

    audio.current.onended = () => {
        setIsPlaying(false);
    }

    audio.current.ontimeupdate = () => {
        setCurrentTime(audio.current.currentTime)
    }

}, [])
```

Don't worry if you don't understand some of the code - I will explain what it does.

First of all we set up our connection with the worker. `worker.port.start();` ensures that our port is open and can receive/send messages.

Next we create an interval using `setInterval` that gets run every second. We use this interval to ping our worker to let it know that our tab has not been closed. For example if a tab gets closed, it will stop pinging the worker, and the worker will assume that the tab was closed and, if the tab was the main tab, will change the main tab to a different one.

We add a message handler to `worker.port.onmessage` to be able to receive messages from the worker. There will only be 2 messages - `set_main_port`, which notifies the tab that it should be the main tab, and `unset_main_port`, which notifies the tab that it should stop being the main tab.

Next up we add some event listeners to our `audio` element. We add an `ondurationchange` listener which updates our `maxTime` state whenever the length of the song changes. Then we add an `onended` listener which sets our `isPlaying` state to false when the playback has ended, and finally we add an `ontimeupdate` listener which updates our `currentTime` state whenever the current time of the `audio` element changes.

Ok, great, we have set up our initialization logic! We're about halfway done with this file!

Our next `useEffect` will be as follows: 

```
useEffect(() => {
        if(currentTime != 0 && currentTime != null) {
            setStoredTime(currentTime);
        }
}, [currentTime])
```

What this simply does is update our `storedTime` variable and store it in localStorage whenever `currentTime` changes.

Now let's write the `useEffect` for when the `src` state changes

```
useEffect(() => {
    if(!audio.current.src || audio.current.src == '' && storedTime != 0) {
        updateTime(storedTime)
    } else {
        setStoredTime(0);
        updateTime(0);
    }
    
    if(src != "") {
        audio.current.src = src;
    }

    if(isPlaying && isMainTab) {
        audio.current.play();
    }
}, [src])
```

First of all, we check if our `src` is defined and if `storedTime` is not 0, in which case we update our current time in accordance to the `storedTime`. This will only ever happen when the page first opens, and will allow us to persist our current time between page refreshes. 

Next we check if `src` is not an empty string, in which case we set the `audio` element's `src` property to the new `src`.

Finally we check if the player should be playing and if the current tab is the main tab, in which case we have our `audio` element play our song.

```
useEffect(() => {
    audio.current.currentTime = updatedTime;
    setCurrentTime(updatedTime);
}, [updatedTime])
```

Next we have a `useEffect` that runs when `updatedTime` changes. We use this `updatedTime` state as a proxy to set our `currentTime` because we cannot set `currentTime` directly. I'll be honest, I forgot the exact reason why we can't set `currentTime` directly, but I know that it can cause an infinite loop when syncing tabs and using the slider to change our current time.

Additionally we update the `currentTime` property of the `audio` element, which makes it playback at that time.

```
useEffect(() => {
    if (isPlaying && isMainTab) {
        // console.log('is main')
        audio.current.play();
    } else {
        audio.current.pause();
    }
}, [isPlaying])
```

Our final `useEffect` will be run when `isPlaying` changes. In it, we check if `isPlaying` is set to true and if the current tab is the main tab, in which case we play the audio. Otherwise we set the audio on pause.

Finally, let's return all the relevant functions and state:

```
return {
    isPlaying,
    setIsPlaying,
    setCurrentTime,
    currentTime,
    updateTime,
    maxTime,
    setSrc
}
```

And we're done with our `useMusicPlayer` hook! 

If you want to see what the final file looks like, refer to [the file in the source code](https://github.com/Printy-Studios/react-music-player/blob/main/src/useMusicPlayer.ts)

Now let's create our `worker.ts` file

## `worker.ts`

Our worker will manage the information about the current main tab, and determine which tab should be the main tab. Depending on your configuration, the `worker.js` file should go either into the `src` folder or the `public` folder. In my case I put it into `public`.

Our worker will look like this:

```
const ports = [];
let main_port = null;
let pinged = true; // Did Main port ping?

const postMessageAll = (message, exclude_port) => {
    ports.forEach((port) => {
        if(port != exclude_port) {
            port.postMessage(message)
        }
        
    })
}

const setMainPort = (port) => {
    main_port = port;
    main_port.postMessage({
        type: "set_main_port"
    })
    postMessageAll({
        type: "unset_main_port"
    }, main_port);
    
}

// Ping check
setInterval(() => {
    if(!pinged) {
        const port_index = ports.findIndex(_port => _port == main_port);
        ports.splice(port_index, 1);
        setMainPort(ports[0])
    }
    pinged = false;
}, 2000);

onconnect = (e) => {
    const port = e.ports[0];

    ports.push(port);

    if(ports.length == 1) {
        setMainPort(port)
    }    

    port.onmessage = (e) => {
        const data = e.data;

        switch(data.type){
            case "ping": {
                if(port == main_port) {
                    pinged = true;
                }
                
            }
        }
    }
}
```

First we define some variables that our worker will need to use. `ports` stores a list of all our open ports, `main_port` stores a reference to our main port/tab, and `pinged` tells us whether main port has successfully pinged the worker.

Next we create a utility function `postMessageAll()` which posts a message to all of the ports. Optionally you can pass a port as the second arg, which will exclude that port from being messaged.

Then we create a function `setMainTab` which sets our main tab - it sends a message to the new main tab to let it know that it is now the main tab, and also sends a message to all the other tabs to let them know that they are not the main tab.

Next up we create an interval that checks every 2 seconds whether the main tab has successfully pinged our worker without a timeout. If the main tab failed to ping the worker, then the worker assumes that the main tab was closed and sets another tab as the main one.

Finally, in `onconnect` we add some initialization logic - if there is only 1 port open, then it should be set to the main port. And we also listen for the `ping` message from the tabs to make sure that the main tab has pinged the worker and hasn't been closed.

That's it for our worker!

Next up let's create our utility functions in `util.ts`

## `util.ts`

Our `util.ts` file will contain some reusable utility functions that will make our code a bit more readable.

First of all is the `secondsToMinutesAndSeconds()` function:

```
export function secondsToMinutesAndSeconds(time: number) {
    const minutes = Math.floor(time / 60);
    const seconds = Math.floor(time - minutes * 60);

    return {
        minutes,
        seconds
    }
}
```

What it does is simply convert seconds to an object storing seconds and minutes.

Next up is `numberToPercent()`:

```
export function numberToPercent(n: number, max_n: number) {
    const percentage = n == 0 ? 0 : (n / max_n) * 100;
    return percentage
}
```

What this does is tell you how much percent is `n` of `max_n`.

Finally let's add `percentOf()`:

```
export function percentOf(n: number, percentage: number) {
    return n * percentage / 100
}
```

This function tells us how much is `percentage` of `n`

And our `util.ts` file is done!

## `SongMetadata.ts`

The `SongMetadata.ts` will hold the type for our song metadata object, it's quite simple:

```
type SongsMetadata = {
    id: string,
    title: string,
    author: string
}

export default SongsMetadata;
```

As you can see it just holds some basic info about the song such as its ID, title and author.

At this point, functionality wise our player is done - we could progammatically invoke this hook and use its functions to play our audio! All that is left is to implement the UI!

Let's start with the small components first

## `IconButton.tsx`

This will be the component for our icon buttons. It will have some basic animations that you'll see once we get to the `style.css` file.

```
type IconButtonProps = {
    src: string,
    onClick: () => void,
}

export default function IconButton({ onClick, src }: IconButtonProps) {
    return (
        <div
            onClick={onClick}
        >
            <img className='icon' src={src} />
        </div>
    )
}
```

As you can see it's basically just a button wrapper around an image with an `icon` class. the `src` prop tells us the image url, and `onClick` is the click handler.

# `PlayButton.tsx`

The `PlayButton` component will just be a basic wrapper around `IconButton` with specific icons that will change depending on the `playing` prop:

```
import IconButton from './IconButton'

type PlayButtonProps = {
    playing: boolean,
    onClick: () => void,
}

export default function PlayButton({ onClick, playing }: PlayButtonProps) {
    return (
        <IconButton
            onClick={onClick}
            src={playing ? 'icons/pause.svg' : 'icons/play.svg'}
        />
    )
}
```

# `TimeDisplay.tsx`

Time display will be a component that will take a `time` (an object of seconds and minutes)` prop and display this time.


type TimeDisplayProps = {
    time: {minutes: number, seconds: number};
}

export default function TimeDisplay({ time }: TimeDisplayProps) {

    return (
        <div className='time-display'>
            {time.minutes}
            :
            {time.seconds.toString().padStart(2, '0')}
        </div>
    )
}

Alright we've made it this far, we only have 1 component left to go - the `MusicPlayer` component itself.

## `MusicPlayer.tsx`

The `MusicPlayer` component will be the component that will use the `useMusicPlayer` hook and the sub-components that we created to display a fully functioning music player.

Let's start by scaffolding our component:

```
export default function MusicPlayer() {

}
```

Now let's start first by adding our state.

```
    const musicPlayer = useMusicPlayer();

    const [sliderValue, setSliderValue] = useState(0);
    const [isSliderActive, setIsSliderActive] = useState(false);

    const [currentTime, setCurrentTime] = useState({minutes: 0, seconds: 0});
    const [maxTime, setMaxTime] = useState({minutes: 0, seconds: 0});

    const [currentSong, setCurrentSongLocal] = usePersistState<number>(0, 'current_song_index');
    const setCurrentSong: (new_state: number) => void = useTabStateControl(currentSong, setCurrentSongLocal, 0, 'current_song_index')
    const [songsMetadata, setSongsMetadata] = useState<SongsMetadata[]>([]);
```

`musicPlayer` is the `useMusicPlayer` hook that we wrote earlier.

`sliderValue` and `isSliderActive` are states that we will use to track and manage our time slider

`currentTime` and `maxTime` are the same as their equivalent states in `useMusicPlayer`, except they've been converted to seconds/minutes objects.

`currentSong` is the state that will track the index number of the current song in the `songsMetadata` array. You can see that we have 2 lines for defining `currentSong` and `setCurrentSong`. First we create the local state with `usePersistState` in order to have the state be stored to localStorage, then we use `useTabStateControl` to additionally sync the state across tabs. `useTabStateControl` is another hook that I wrote that allows you to combine a state handler with `useTabState`. So you can have both persistant state and tab synced state this way. Another cool thing (and the initial reason I made `useTabStateControl`) is that this way you can hook up any state management library, like Redux or Zustand to the `useTabState` hook.

`songsMetadata` holds an array of the available songs.

Now let's start writing the functions that the component will use.

```
const onPlayButtonClick = () => {
    musicPlayer.setIsPlaying(!musicPlayer.isPlaying)
}
```

As the name suggests, this function will handle the clicking of the play button, and simply toggle the music player's `isPlaying` state.

```
const onPrevButtonClick = () => {
    if(currentSong == 0) {
        setCurrentSong(songsMetadata.length - 1);
    } else {
        setCurrentSong(currentSong - 1);
    }
}

const onNextButtonClick = () => {
    if(currentSong == songsMetadata.length - 1) {
        setCurrentSong(0);
    } else {
        setCurrentSong(currentSong + 1);
    }
}
```

These 2 functions will handle the clicking of the previous/next buttons. What they do is simply increment or decrement the `currentSong` index by 1, or have it loop to the first/last index.

```
const fetchSongsMetadata = () => {
    fetch('songs_metadata.json')
    .then(res => res.json())
    .then(setSongsMetadata);
}
```

This function simply fetches our songs metadata file and adds the returned array to the `songsMetadata` state

```
const onSliderChange = (e: ChangeEvent<HTMLInputElement>) => {
    setSliderValue(parseInt(e.currentTarget.value));
    setIsSliderActive(true);
}

const onSliderMouseUp = () => {
    const new_time = percentOf(musicPlayer.maxTime, sliderValue);
    musicPlayer.updateTime(new_time)
    setIsSliderActive(false);
}
```

These 2 functions handle the slider. `onSliderChange` gets called every time our slider gets dragged around, and it updated our `sliderValue` as well as marks the slider as being active (in dragging state). We don't want to update our songs current time every time the slider value changes(this would result in a jagged sound when the slider gets moved around), but only once it has been released. For this we have the `onSliderMouseUp` function, which gets called whenever the slider handle has been released. The function updates our music player's current time, and marks the slider as not being dragged anymore.

```
const loadSong = (index: number) => {
    if(songsMetadata.length) {
        musicPlayer.setSrc(`songs/${songsMetadata[index].id}.wav`)
    }
}
```

The `loadSong` function simply loads the songs at the given index of `songsMetadata` array, by setting the music player's `src` to that song's source url.

Next up, let's add our `useEffect`s

```
useEffect(() => {
    fetchSongsMetadata()
}, [])

useEffect(() => {
    loadSong(currentSong);
}, [songsMetadata])
```

These  2 `useEffects` make sure that the song metadata gets downloaded and that the initial song gets loaded.

```
useEffect(() => {
    if(!isSliderActive) {
        if(!musicPlayer.currentTime || !musicPlayer.maxTime) {
            setSliderValue(0)
        } else {
            setSliderValue(numberToPercent(musicPlayer.currentTime, musicPlayer.maxTime));
        }
        
    }
    setCurrentTime(secondsToMinutesAndSeconds(musicPlayer.currentTime));
}, [musicPlayer.currentTime, musicPlayer.maxTime])
```

This `useEffect` updates the slider value to the current time of the song, so you can see at which point in time the song is currently playing. We also make sure that the slider value doesn't get updated while the slider is being dragged (because this would result in buggy behavior).

```
useEffect(() => {
    setMaxTime(secondsToMinutesAndSeconds(musicPlayer.maxTime));
}, [musicPlayer.maxTime])
```

Finally we has a `useEffect` for the music player's `maxTime` which updates the component's `maxTime` accordingly.

Alright, we're almost done! Now all that is left is to build out the markup and css! This is what the markup looks like: 

```
return (
    songsMetadata.length > 0 ?  
    <div className='music-player'>
        <div className='music-player-info'>
            <img 
                src={`cover_art/${songsMetadata[currentSong].id}.jpg`} 
                width='100%'
            />
            <h5>{songsMetadata[currentSong].title}</h5>
            <h6>{songsMetadata[currentSong].author}</h6>
        </div>
        
        <div className='music-player-controls'>
            <IconButton
                onClick={onPrevButtonClick}
                src={'icons/prev.svg'}
            />
            <PlayButton onClick={onPlayButtonClick} playing={musicPlayer.isPlaying} />
            <TimeDisplay time={currentTime} />
            <input 
                type="range" 
                min="0" 
                max="100" 
                value={sliderValue} 
                className="slider" 
                onMouseUp={onSliderMouseUp}
                onChange={onSliderChange}
            />
            <TimeDisplay time={maxTime} />
            <IconButton
                onClick={onNextButtonClick}
                src={'icons/next.svg'}
            />
        </div>
    </div>
    : null
)
```

First of all we make sure to render the music player only if we have any songs downloaded.

Next up we have the `music-player-info` element which shows the cover art of the song, the song's title and artist.

Below that we have `music-player-controls` which holds the control elements of our player - the prev/next/play buttons and the slider.

The first `IconButton` component is the prev button, and we make sure to hook up the appropriate event handler.

Next is the `PlayButton` component, where we also make sure to add the event handler.

Then we have a `TimeDisplay` component which displays the current time of the song.

Then we have our slider element which uses the native HTMLS range input.

Next we have the time display for our `maxTime`

And finally we have our 'next' button.

We're so close! All that is left now is to add the necessary styles!

## `style.css`

This is what our CSS looks like:

```
body, html {
  font-family: Inter, system-ui, Avenir, Helvetica, Arial, sans-serif;
  
  margin: 0px;

  height: 100vh;
  width: 100%;
}

#root {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  padding: 8px;
  box-sizing: border-box;
}



.player-wrapper {
  max-width: 400px;
  width: 100%;
}

/* Icons */

.icon {
  width: 32px;
  height: 32px;

  cursor: pointer;

  transition: transform 0.1s ease-in-out;
}

.icon:hover {
  transform: scale(105%);
}

.icon:active {
  transform: scale(70%);
}

/* Music player */

.music-player {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.music-player-controls {
  display: flex;
  align-items: center;
  gap: 8px;
}

/* Music player info */

h5 {
  font-size: 16px;
  margin: 0px;
}

h6 {
  font-size: 14px;
  margin: 0px;
  font-weight: normal;
  color: gray;
}

/* Time display */

.time-display {
  margin-bottom: 2px;
  color: gray;
}

/* Slider */

.slider {
  -webkit-appearance: none;
  flex-grow: 1;
  height: 4px;
  background: #d3d3d3;
  border-radius: 100px;
  outline: none;
  opacity: 0.7;
  -webkit-transition: .2s;
  transition: opacity .2s;
  min-width: 0px;
}

.slider:hover {
  opacity: 1;
}

.slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 25px;
  height: 25px;
  border-radius: 100px;
  background: #04AA6D;
  cursor: pointer;
}

.slider::-moz-range-thumb {
  width: 25px;
  height: 25px;
  border-radius: 100px;
  background: #04AA6D;
  cursor: pointer;
}
```

I won't be going into details about how the CSS works because I don't think I'd be able to explain it haha. But if you have any questions about any of the code feel free to ask and I will try my best to explain!

Aaaaaand we're done with our music player!

# `App.tsx`

Finally, in our `App.tsx`, we will add the following code to display our music player component:

```
import MusicPlayer from './MusicPlayer';


function App() {

  return (
    <div className='player-wrapper'>
      <MusicPlayer />
    </div>
  );
}

export default App
```

Now you can go ahead and test your music player and see that it(hopefully) works! You can even add your own songs! Just add the appropriate files (`.wav` for the song itself, `.jpg` for the cover art and then add the song info to the `songs_metadata.json`)

## Conclusion

Whew, this was a long one! The longest article I've ever written! If you read through the whole post then congratulations! 

In this post we explored a way on how to build a tab-synced and persistant music player in React. If you're interested in how `useTabState` and `usePersistState` work, check out the articles linked below:

 * [How to keep state between page refreshes in React](https://dev.to/jorensm/how-to-keep-state-between-page-refreshes-in-react-3801)
 * [How to sync React state across tabs with workers](https://dev.to/jorensm/how-to-sync-react-state-across-tabs-with-workers-2mpg)
 * 
I hope you learned something new from this article. If you have any questions or feedback, feel free to leave a comment!

If you're interested in more web dev content, consider subscribing to my [bi-weekly newsletter](http://eepurl.com/hTqj4j). There I share with you top web dev articles that I've recently found on the internet.

If you have any requests or ideas for an article, feel free to share them with me in the comments or at [jorensmerenjanu@gmail.com](mailto:jorensmerenjanu@gmail.com).

Have a great day!


Tags:
  Article, Blog, React
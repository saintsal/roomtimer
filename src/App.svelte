<script>
  import { onDestroy, onMount } from 'svelte';
  import Menu from './Menu.svelte';

  //Interface
  let
    timeRequest = decodeURI(window.location.pathname.slice(1)),
    menuOpen = timeRequest.length>0 ? false : true,
    timeInput,
    isMuted = false,
    isFS = timeRequest.length>0 ? true: false,
    mode=0;
 
  let modes= [
    {sound: "bell.mp3", cssclass:"normal" },
    {sound: "alright.mp3", cssclass:"mm" } //Mathew!
  ];

  //Timer function
  let 
    startTS = window.performance.now(),
    //last_time = window.performance.now(),
    frame,
    elapsed = 0,
    timeLeft = 0,
    duration = 0,
    elapsedSaved = 0,
    running = timeRequest.length>0 ? true: false,
    bellPoints = new Array(),
    timers = [{time:0, label: "0:00"}],
    timerMarker = 0;

  //Display
  let
    percentage = 0,
    curLabel = "",
    isFinal = false,
    isMMmode = false,
    bar,
    app;


  $: curLeft = curDuration-curElapsed;
  $: curElapsed = elapsed-prevTimers;
  $: isFinal = running && (timeLeft < Math.min(duration/5, 60000));
  $: endTS = startTS + duration * 1000;
  $: percentage  = Math.min(100,(Number(duration-timeLeft)/ Number(duration)*100).toFixed(2));
  $: curPercentage = timeLeft > 0 ? Math.min(100,(Number(duration-timeLeft-prevTimers)/ curDuration*100).toFixed(2)): 100;
  $: upcomingBell = bellPoints[timerMarker];
  $: { parseTimeRequest(); timeRequest; }
  $: curDuration = (typeof timers[timerMarker] !== 'undefined') ? timers[timerMarker].time : duration;
  $: prevTimers = (typeof bellPoints[timerMarker-1] !== 'undefined')?  bellPoints[timerMarker-1]: 0;
  $: displayedMessage = curLabel ? curLabel : MStoClock(curLeft);

  onMount(async () => {
    parseTimeRequest();
    startTicker();
    if (menuOpen) timeInput.focus();
  });
  
  function ticker() {
    frame = requestAnimationFrame(ticker);

    if (running) {
      //Increment elapsed
      const time = window.performance.now();
      elapsed = time - startTS;

      //Using a Svelte declaration ($: ) doesn't update fast enough so we update timestamp-based vars here
      timeLeft = duration - elapsed;
      curLabel = timers[timerMarker].label;

      //Next timer
      if (elapsed > upcomingBell) {
	playBell();
	timerMarker++;
      }
    
      //End the timer
      if (timeLeft < 0) {
	running = false;
	isFinal = false;
	curLabel = "Done!";
	playBell();
	//parseTimeRequest(); //Reset the timers array to avoid errors and setup space to restart the same timer
      }
    }
  }

  /* Behaviour */
  function startTicker() {
    startTS = window.performance.now();
    ticker();

    onDestroy(() => {
      cancelAnimationFrame(frame);
    });

  }

  function restart() {
    parseTimeRequest();
    startTS = window.performance.now();
    elapsed = 0;
    isFinal = false;
    running = true;
  }

  function parseTimeRequest() {

    var totaltime = 0;

    //clear
    timers = [{time:0, label: "0:00"}],
    bellPoints = new Array ();
    timerMarker = 0;

    if (timeRequest) {
      var p = timeRequest.split('/');
      if (timeRequest.includes("/")) {
	for (var i = 0; i < p.length; i++) {
	  timers[i] = {
	    time: convertClockToSeconds(p[i]),
	    label:  convertClockToLabel(p[i])
	  }
	  totaltime += timers[i].time;
	  bellPoints[i] = totaltime ;
	}

      } else {
	timers[0] = {
	  time: convertClockToSeconds(timeRequest),
	  label: convertClockToLabel(timeRequest)
	}
	totaltime = timers[0].time;
      }
	
      duration = totaltime;
      timeLeft = totaltime;

    }
  }

  function handleMenu(event) {
    switch (event.detail.open) {
      case true:
	timeInput.focus()
	timeInput.select();
	break;
      case false:
	timeInput.blur(); //Don't want to have the user typing into the input if they can't see it
	break;
    }
  }

  function handleInputKey(event) {
    let key = event.key;
    let keyCode = event.keyCode;
  
    switch (keyCode) {
      case 13: //Enter
	restart();
	menuOpen = false;
	break;
    }
  }

  function fullscreen() {
    isFS = !isFS;
    if (isFS) {
      app.requestFullscreen();
    } else {
      document.exitFullscreen();
    }
  }

  function mute() {
    isMuted = !isMuted;
  }

  function beastmode() {
    mode = (mode + 1) % modes.length;
    for (var m of modes) {
      bar.classList.remove(m.cssclass);
    }
    bar.classList.add(modes[mode].cssclass);
  }

  function pause() {
    if (timeLeft > 0) {
      if (running) {
	elapsedSaved = elapsed;
      } else {
	startTS = window.performance.now() - elapsedSaved;
	elapsedSaved = 0;
      }
      running = !running;
    } else {
      restart();
    }
  }

  function moreTime(seconds) {
    const ms = seconds*1000;
    if (running) {
      duration += ms;
      timers.forEach(function(obj, index) {
	if (index >= timerMarker) {
	  timers[index].time += ms;
	  bellPoints[index] += ms;
	}
      });
    }
  }

  function handleKeydown(event) {
    let key = event.key;
    let keyCode = event.keyCode;

    switch (keyCode) {
      case 32: //Space
	pause();
        break;
      case 82: //R
	restart();
        break;
      case 77: //M
	mute();
        break; 
      case 70: //F
	fullscreen();
        break; 
      case 66: //B
	beastmode();
        break; 
      case 84: //t
	moreTime(10);
        break; 
      case 49: //1
	moreTime(60);
        break; 
      case 50: //2
	moreTime(120);
        break;
      case 51: //3
	moreTime(180);
        break;
      case 52: //4
	moreTime(240);
        break;
      case 53: //5
	moreTime(300);
        break;
      case 54: //6
	moreTime(360);
        break;
      case 55: //7
	moreTime(420);
        break;
      case 56: //8
	moreTime(480);
        break;
      case 57: //9
	moreTime(520);
        break;
      case 58: //0
	moreTime(600);
        break;
    }
  }

  function playBell() {
    if (!isMuted) {
      var bell = new Audio("/sounds/"+modes[mode].sound);
      bell.play();
    }
  }

  /* Derived values */
   
    function convertClockToLabel(clock) {
      if (!clock) return false; 

      if(clock = clock.match(/\(([^)]+)\)/)) {
        return clock[1];
      } else {
        return "";
      }
    }

    function convertClockToSeconds(clock) {
      if (!clock) return false; 

      //Strip labels
      clock = clock.replace(/\(([^)]+)\)/gi, '');
      //Strip illegal chars
      clock = clock.replace(/[^[0-9:]/gi, '');

      var p, s, m;

      if (clock.includes(":")) {
        p = clock.split(':'), s = 0, m = 1;
        while (p.length > 0) {
          s += Number(m) * parseInt(Number(p.pop()), 10);
          m *= 60;
        }
      } else {
        s = Number(clock);
      }

      return s*1000;
    }

  

  function MStoClock(value) {
      value /= 1000;

      // timeleft rounds down, so add a second to the way it's displayed
      value += 1;

      var sec_num = parseInt(value, 10); // don't forget the second param
      var hours   = Math.floor(sec_num / 3600);
      var minutes = Math.floor((sec_num - (hours * 3600)) / 60);
      var seconds = sec_num - (hours * 3600) - (minutes * 60);
      var ret;

      if ((minutes < 10 && hours > 0)) {minutes = "0"+minutes;}
      if (seconds < 10) {seconds = "0"+seconds;}

      ret = seconds;
      ret = minutes+ ':' + ret;
      if (hours > 0 ) ret = hours + ':' + ret;
      
      return ret;
  }



</script>


<style>
  :global(body) { position: fixed; padding: 0;}
  :-webkit-full-screen { background: #fff; }
  :-moz-full-screen { background: #fff; }
  :-ms-full-screen { background: #fff; }
  #bottomcontrols { position: fixed; right: 10px; bottom: 10px; z-index: 20}
  #controls { position: absolute; right: 10px; top: 6%; }
  .bellline { position: absolute; top: 0; height: 4%; float: left; border-right: 0.5rem solid #999; z-index:20;}
  #prg { position: absolute; width: 100%; height: 100%; }
  #timer{position: absolute; width: 100%; height: 100%; font-size: 18vw; text-align: center; vertical-align: middle; mix-blend-mode: multiply; font-family: "PT Mono", monospace; }
  #timer span {position: fixed; left: 0; top: 50%; width: 100%; transform: translateY(-50%); font-color: #999;}
  #timer span.label { font-family: "Oxygen"; }
  #footer {position: absolute; bottom: 10px; left: 10px; right: 10px; height: 4vh; font-size: 3vh; text-align: center; mix-blend-mode: multiply; }
  #footer .logo {mix-blend-mode: overlay; }
  #bar { height: 100%; float: left; background: #18c953;  -webkit-transition: 0.1s linear ; -moz-transition: 0.1s linear; -o-transition: 0.1s linear ; transition: 0.1s linear;  }
  #totalbar { position: absolute; height: 4%; float: left; background: #000;  -webkit-transition: 0.1s linear ; -moz-transition: 0.1s linear; -o-transition: 0.1s linear ; transition: 0.1s linear;  }
  .final #bar {    animation-duration: 500ms; animation-name: blink; animation-iteration-count: infinite; animation-direction: alternate;}
  :global(#bar.mm) { background-image: url(/images/mm.gif); background-size: cover; background-position-y: 30%; opacity: 0.7;}
  #help { margin: 40px 30px;}
  .logo {height: 4vh;}
  form {margin: .5rem; text-align: center;}
  #controls img, #fullscreen img { width: 1rem; min-width: 50px; min-height: 50px; height: 1rem; padding: 4px; }
  .timeleft {  -webkit-touch-callout: none; -webkit-user-select: none; -khtml-user-select: none; -moz-user-select: none; -ms-user-select: none; user-select: none; }
  @media all and (max-height: 250px) {
    #timer{font-size: 80vh; }
  }
  @media all and (max-height: 500px) {
    #bottomcontrols img, #controls img { width: 2rem; min-width: 30px; min-height: 30px; height: 2rem; padding: 4px; }
  }
  @keyframes blink { from { opacity: 1; } to { opacity: 0.3; } }

ul {
  list-style: none;
  padding: 0;
}

.menu {
  font-size: 1rem;
}

.menu-list {
  line-height: 1.25;
}

.menu-list a {
  border-radius: 2px;
  color: #4a4a4a;
  display: block;
  padding: 0.5em 0.75em;
}

.menu-list a:hover {
  background-color: whitesmoke;
  color: #363636;
}

.menu-list a.is-active {
  background-color: #3273dc;
  color: #fff;
}

.menu-list li ul {
  border-left: 1px solid #dbdbdb;
  margin: 0.75em;
  padding-left: 0.75em;
}

.menu-label {
  color: #7a7a7a;
  font-size: 0.75em;
  letter-spacing: 0.1em;
  text-transform: uppercase;
}

.menu-label:not(:first-child) {
  margin-top: 1em;
}

.menu-label:not(:last-child) {
  margin-bottom: 1em;
}

</style>



<svelte:head>
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <!--Import Google Icon Font-->
  <!-- <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet"> -->
  <title>{displayedMessage}</title>
</svelte:head>

<svelte:window on:keydown={handleKeydown}/>

<div id="app" allowfullscreen=true bind:this={app} class:final={isFinal} >
  {#each bellPoints as line}
    <div class="bellline" style="width: {line/duration*100}%"></div>
  {/each}
  <div id="prg">
    <div id="bar" bind:this={bar} style=" width: { curPercentage + '%' }"></div>
    <div id="totalbar" style=" width:{ percentage + '%' }"></div>
  </div>
  <div id="timer" on:click={pause}>
    <span class="timeleft">{displayedMessage}</span>
  </div>
  <div id="controls">
      <Menu on:message={handleMenu} bind:isopen={menuOpen} bind:time={timeRequest}>
	<p class="menu-label">
	  Set time
	</p>
	<ul class="menu-list">
	  <li>
	    <input placeholder="Type seconds or HH:MM:SS" autocomplete="off" on:keydown={handleInputKey} bind:value={timeRequest} bind:this={timeInput} />
	  </li>
	</ul>
       <p class="menu-label">
	  Sequential timers
	</p>
	<ul class="menu-list">
	  <li><strong>/</strong> between timers.  Try <strong>3/8/2:30</strong> </li>
	</ul>
	<p class="menu-label">
	  Add Instructions
	</p>
	<ul class="menu-list">
	  <li><strong>()</strong> after each time. eg.<br><strong>2(Ready)/3(Set)/2:30(Dance!)</strong>.</li>

	</ul>
	<p class="menu-label">
	  Autostart
	</p>
	<ul class="menu-list">
	  <li>Add the timer to the address.<br>eg. <strong>sourcetimer.com/3:00</strong></li>    
	  <li>(Great to link from your slides!)</li>
	</ul>
	<p class="menu-label">
	  Control Keys
	</p>
	<ul class="menu-list">
	  <li><strong>Esc</strong> opens this menu</li>
	  <li><strong>Space</strong> pauses</li>
	  <li><strong>R</strong> restarts </li>
	  <li><strong>F</strong> toggles fullscreen</li>
	  <li><strong>M</strong> toggles Matthew McConaughey</li>
	  <li><strong>1-9</strong> adds bonus time (in minutes)
	  <li><strong>0</strong> adds 10 minutes
	  <li><strong>T</strong> adds 10 seconds
	</ul>
      </Menu>
  </div>
  <div id="footer">
    {isMuted} 
    Made by <a href="http://source.institute"><img alt="Source" src="/images/logo.png" class="logo"></a>
  </div>

</div>

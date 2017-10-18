Cutredad


<html lang="en">
<head>
  <link rel="authors" href="/humans.txt" />

  <meta charset="utf-8" />
  <title>Is it Christmas?</title>

  <link rel="shortcut icon" href="/icons/large.png" />

  <link rel="alternate" title="Is It Christmas?" href="/rss.xml" type="application/rss+xml" />
  <link rel="alternate" title="Is It Christmas? (Christmas only)" href="/rss.xml?only=yes" type="application/rss+xml" />

  <meta name="twitter:site" content="@konklone">
  <meta name="twitter:creator" content="@konklone">
  <meta property="og:site_name" content="Is It Christmas?" />
  <meta property="og:type" content="website" />
  <meta property="og:url" content="https://isitchristmas.com" />

  <meta property="og:title" content="Is It Christmas?" />

  <meta name="twitter:card" content="summary">
  <meta property="og:image" content="https://isitchristmas.com/icons/large-wide.png" />

  <style type="text/css">
    html, body {height: 100%;}
    body {text-align: center;}

    a#answer {
      display: inline-block;
      margin-top: 200px;
      font-weight: bold;
      font-size: 120pt;
      font-family: Arial, sans-serif;
      text-decoration: none;
      color: black;
    }

    .flag {
      position: absolute;
      cursor: none;

      border: 1px solid #d6d6d6;

      -webkit-border-radius: 2px;
           -o-border-radius: 2px;
              border-radius: 2px;
    }
    .flag.ghost {opacity: 0.4;}
    .flag.me {pointer-events: none;}

    .click {
      position: absolute;
      border: 1px solid #000;

      -webkit-border-radius: 3px;
              border-radius: 3px;

      -webkit-transition: 0.5s ease-out;
         -moz-transition: 0.5s ease-out;
           -o-transition: 0.5s ease-out;
              transition: 0.5s ease-out;
    }

    #legend {
      position: fixed;
      top: 0; right: 0;
      width: 200px;
      padding-right: 15px; padding-top: 5px;
      text-align: right;
      font-size: 10pt;
    }

    /* remove display: none, and turn on setTimers(), to restore */
    #links {
      position: fixed;
      top: 0; left: 0;
      width: 300px;
      padding-left: 15px; padding-top: 5px;
      text-align: left;
      font-size: 10pt;
      z-index: 9999;

      opacity: 0;

      display: none;

      -webkit-transition: 0.5s ease-out;
         -moz-transition: 0.5s ease-out;
           -o-transition: 0.5s ease-out;
              transition: 0.5s ease-out;
    }

    #links a {
      color: #333;
      display: inline-block;
      margin-right: 10px;
    } #links a#console {display: none;}
    /* 4 9's, needs to be clickable */
  </style>

  <script type="text/javascript" src="/js/christmas.js"></script>

  <!-- IE11 JS targeting, from http://stackoverflow.com/a/17447695 -->
  <script type="text/javascript">
    window._ie11 = !!navigator.userAgent.match(/Trident\/7.0; rv 11/);
    window._edge = !!navigator.userAgent.match(/Edge/);
    window._ie = window._ie11 || window._edge;
    window._firefox = !!navigator.userAgent.match(/Firefox/);
  </script>
</head>

<body>
  <!--
    Initial 'title' and noscript values are server-side fallbacks,
    calculated with UTC, for clients who do not have JS enabled.
  -->
  <a id="answer"
    href="https://ifttt.com/isitchristmas"
    target="_blank"
    title="IFTTT">
    <noscript>NO</noscript>
  </a>

  <!-- fade in after a couple seconds -->
  <div id="links">
    <a href="https://twitter.com/konklone" target="_blank">by @konklone</a>
    <a href="https://ifttt.com/isitchristmas" target="_blank">on IFTTT</a>
    <!-- filled in dynamically by JavaScript -->
    <a id="console" href="#" target="_blank">console</a>
  </div>

  <div id="legend"></div>

  <!-- replace fallback data with locally calculated values -->
  <script type="text/javascript">
    var country = "ES";
    if (!(Christmas.countries[country] && Christmas.countries[country].width))
      country = "EO";

    var me = {
      country: country
    };

    var checkedAt; // store last check, to manage race conditions
    function updateChristmas(isIt) {
      me.christmas = isIt;

      var answer;
      if (isIt)
        answer = Christmas.yes(country);
      else
        answer = Christmas.no(country);

      var elem = document.getElementById('answer');
      elem.innerHTML = answer;
      elem.setAttribute("title", answer);

      checkedAt = Date.now();
    }

    updateChristmas(Christmas.isIt());
  </script>


  
    <script>
      (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
      (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
      m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
      })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');

      ga('create', 'UA-252618-5', 'isitchristmas.com');
      ga('set', 'anonymizeIp', true);
      ga('set', 'forceSSL', true);
      ga('send', 'pageview');

    </script>
  


  <!-- optional: interaction -->
  <script src="/js/browser.js"></script>
  <script src="/js/css.js"></script>
  <script src="/js/sockjs.min.js"></script>
  <script src="/js/emoji.js"></script>
  <script src="/js/api.js"></script>
  <script type="text/javascript">

    var socket;
    var others = {};

    var user = {
      log: {
        join: false, // join/leave messages
        info: false,   // default
        debug: false, // you know

        life: true,  // life and death events
        chat: true,  // chat messages
        system: true // messages from the system for the public
      },

      retry: {
        id: null, // the timeout id
        initial: 500, // milliseconds
        multiplier: 2, // multiply after every retry
        current: 500 // will change
      },

      // values will be overwritten on hello in production
      live: {
        mouse_rate: 20,
        heartbeat_interval: 3000,
        death_interval: 6000,
        chat: "true",

        ghost_duration: 2000,
        ghost_max: 10
      }
    };

    me.flag = null;
    me.ghosts = 0;
    me.angle = 0;
    me.alreadyArrived = false;
    me.notify = false;

    if (window.localStorage) {
      me.savedName = window.localStorage.getItem("me.name");
      me.notify = (window.localStorage.getItem("me.notify") == "true");
      if (canNotify && me.notify && !(Notification.permission == "granted"))
        notifications.permission();
    }

    // sockjs server endpoint
    
      
      var url = "/christmas";
    

    // whether activity is visible to the user
    var visible = true;

    // whether we need to force chat to stay plain-text (no styles).
    // right now, the only situation that's relevant is IE.
    var plainConsole = false || !!window._ie;

    var backgroundImages = !(plainConsole || window._firefox);

    // force it to use a particular transport
    var transport = "";

    // we only need websockets these days (huzzah!)
    var default_transports = ["websocket"];

    // html5 notifications (opt-in)
    var canNotify = "Notification" in window;

    // tiny events/commands system
    var events = {};
    var commands = {};
    function on(event, func) {events[event] = func;}
    function command(cmd, func) {commands[cmd] = func;}
    function noop() {};

    // managing heartbeats
    var heartbeat, death;
    var suicide = false; // why would you do this

    // country legend and fade timer
    var legend = document.getElementById("legend"),
        legendTimeout;

    // timers for xmas-in and xmas-out
    var christmasTimer,
        christmasEndTimer;

    // display a custom console link per-browser
    var consoles = {
      firefox: "https://developer.mozilla.org/en-US/docs/Tools/Web_Console#Opening_the_Web_Console",
      chrome: "https://developer.chrome.com/devtools/docs/console#basic-operation",
      explorer: "https://go.microsoft.com/fwlink/p/?LinkID=309077"
    }

    var clickMap = {0: "left", 1: "middle", 2: "right"};
    var clickbacks = {};


/**** basic connection and event management ***/

    function connect(host) {
      socket = new SockJS(host, null, {
        protocols_whitelist: transport ? [transport] : default_transports
      });

      socket.onopen = function() {
        log.info("= Connected via " + socket.protocol);

        me.transport = socket.protocol;

        // reset retry timer, we're in
        user.retry.current = user.retry.initial;
      };

      socket.onclose = onDeath;

      socket.onmessage = function(message) {
        var data = JSON.parse(message.data);
        (events[data._event] || noop)(data);
      };
    }

    var rawSend = function(message) {socket.send(message)};
    var limiters = {
      chat: ratelimit(rawSend, 500),
      motion: ratelimitLive(rawSend, 'mouse_rate'),
      scroll: ratelimit(rawSend, 50),
      click: ratelimit(rawSend, 50)
      // covers click, scroll, anything else
      // fallback: ratelimit(rawSend, 20)
    };

    function emit(event, data) {
      if (!me.id || !me.transport) {
        log.info("= Weird state! Trying to emit events when I have no ID. :|")
        return;
      }

      data = data || {};
      data._event = event;
      var message = JSON.stringify(data);
      (limiters[event] || rawSend)(message);
    }



/*** user join/leave management ***/

    on('hello', function(data) {
      log.info("= Assigned ID: " + data.user.id + " [on: " + data.server + "]");

      // used only directly by other clients when packets are blindly rebroadcast.
      // me.id will never be depended on by the server.
      me.id = data.user.id;

      // convenience only for console poking, can safely delete
      me.server = data.server;

      // for display only, never sent, is overwritten on server-validated rename
      me.name = data.user.name;

      // server-overridden client options
      for (var key in data.live)
        user.live[key] = data.live[key];

      me.browser = BrowserDetect.browser;
      me.os = BrowserDetect.OS;
      me.version = BrowserDetect.version;

      // welcome user, unless it's a reconnect
      if (!me.alreadyArrived)
        welcomeUser();

      // if we have a saved name, re-validate that name with the server
      if (me.savedName)
        rename(me.savedName);
      else {
        myName();
        log.system(" ");
      }


      // all users announce their info to the server and start a heartbeat
      emit('arrive', myHeart());

      // update me to indicate I've sent 'arrive' once
      // server will know that future connects are reconnects
      me.alreadyArrived = true;

      setHeartbeat();
      setCursor(me.country);

      document.onmousemove = mouseMove;
      document.onmousedown = mouseClick;
      document.onmousewheel = mouseWheel;
      document.onkeydown = keyDown;
      document.addEventListener("DOMMouseScroll", mouseWheel, false);
      document.oncontextmenu = function() {return false};
    });

    // core user details
    var myHeart = function() {
      return {
        id: me.id, // used (and validated) in rebroadcasting only
        angle: me.angle, // useful on reconnecting
        alreadyArrived: me.alreadyArrived, // new or reconnect?
        country: me.country, // this is used only on arrival
        transport: me.transport,
        browser: me.browser,
        version: me.version,
        os: me.os
      };
    };

    var onDeath = function() {
      log.life("= Disconnected! :(");

      clearTimeout(heartbeat);
      clearTimeout(death);

      for (var id in others)
        removeOther(id);

      document.onmousemove = null;
      document.onmousedown = null;
      document.onmousewheel = null;
      document.oncontextmenu = null;
      document.onkeydown = null;

      me.id = null;
      me.transport = null;
      me.time = null;
      if (me.flag && me.flag.parentElement)
        me.flag.parentElement.removeChild(me.flag);

      setRetry();
    };

    var setRetry = function() {
      log.life("= Retrying in " + user.retry.current + "ms...");
      user.retry.id = setTimeout(function() {
        connect(url);
      }, user.retry.current);

      user.retry.current = user.retry.current * user.retry.multiplier;
    }

    var setHeartbeat = function() {
      clearTimeout(heartbeat);
      heartbeat = setTimeout(function() {
        log.debug("heartbeat: beating");
        emit("heartbeat", myHeart());
      }, parseInt(user.live.heartbeat_interval));

      death = setTimeout(function() {
        log.life("= Died from lack of heartbeat :(")
        socket.close();
      }, parseInt(user.live.death_interval));
    };

    on("heartbeat", function(data) {
      if (suicide) return; // let death take me

      clearTimeout(death); // death averted
      log.debug("heartbeat: returned, death averted");

      setHeartbeat();
    });

    on('arrive', function(other) {
      registerOther(other);

      // let a new arrival know you are already here
      emit("here", {to: other.id});
    });

    on('here', function(other) {
      registerOther(other);
    });

    on('leave', function(data) {
      removeOther(data.id);
    });

    on('ping', function(data) {
      log.debug("Ping.");
      if (me.flag) {
        var x = parseInt(me.flag.style.left);
        var y = parseInt(me.flag.style.top);
        if (!isNaN(x) && !isNaN(y))
          emit("pong", {x: x, y: y, angle: me.angle});
      }
    })

    var registerOther = function(other) {
      if (others[other.id]) return;

      others[other.id] = {
        country: other.country,
        ghosts: 0,
        angle: (other.angle || 0)
      };

      var flag = "%c[" + other.country + "]";
      log.join(flag + "%c [" + other.id.slice(8) + "] Joined from " + other.country + " (" + Christmas.countries[other.country].name + ")", styles.flag(other.country), styles.join);
    };

    // the creation of an element per-user is only sparked by that user's mouse motion.
    // so read-only (non-WS) clients will connect and be known, but not cause
    // elements to be generated.
    var showOther = function(other) {
      // make up for shortened key name
      other.country = other.country || other.c;

      // in case this comes before the 'arrive' event does
      registerOther(other);

      others[other.id].flag = flagFor(other.country, "other");

      if (others[other.id].angle)
        setRotate(others[other.id].flag, others[other.id].angle);

      if (visible)
        document.body.appendChild(others[other.id].flag);
    };

    var removeOther = function(id) {
      if (!others[id]) return;
      var country = others[id].country;

      var elem = others[id].flag;
      if (elem && elem.parentElement)
        elem.parentElement.removeChild(elem);

      delete others[id];
      var flag = "%c[" + country + "]";
      log.join(flag + "%c [" + id.slice(8) + "] Departed from " + country + " (" + Christmas.countries[country].name + ")", styles.flag(country), styles.join);
    };


/**** FLAG MECHANICS ***/

    var setCursor = function(country) {
      me.flag = flagFor(country, "me");
      me.flag._new = true; // used to lazy-add element only if moved
      me.flag.style.zIndex = 999; // 3 9's, on top of the world
      setRotate(me.flag, me.angle); // preserve angle on reconnect
    };

    var setRotate = function(elem, angle) {
      if (!elem) return;
      elem.style.transform = elem.style.webkitTransform = elem.style.msTransform = "rotate(" + angle + "deg)";
    };

    var flagFor = function(country, klass) {
      var div = document.createElement('div');
      div.className = "flag " + klass;

      var flag = document.createElement('img');
      flag.src = "/countries/" + country + ".png";

      // create a legend in the top-right when you mouse over other people's flags
      if (klass == "other") {
        div.onmouseover = function() {
          clearTimeout(legendTimeout);
          legend.innerHTML = Christmas.countries[country].names.join("<br/>");
          legend.style.opacity = 1;
          legendTimeout = setTimeout(function() {
            legend.style.opacity = 0;
          }, 20000)
        };
      }

      div.appendChild(flag);

      // without this, div adds height padding, weird
      div.style.height = "20px";

      div.style.marginTop = "-10px"; // half of constant height 20
      div.style.marginLeft = "-" + (flagWidth(country) / 2) + "px";

      return div;
    };

    // there is sadly no international standard on flag aspect ratio
    var flagWidth = function(country) {
      return ((Christmas.countries[country] && Christmas.countries[country].width) || 40);
    }


    var mouseMove = function(event) {
      event = event || window.event;

      // only show it on first movement
      if (me.flag._new && visible && event.clientX && event.clientY) {
        document.body.appendChild(me.flag);
        document.body.style.cursor = "none";
        document.getElementById("answer").style.cursor = "none";
        me.flag._new = false;
      }

      moveFlag(me.flag,
        event.clientX + window.pageXOffset,
        event.clientY + window.pageYOffset
      );

      // is quick-rebroadcast, no server processing
      emit('motion', {
        x: event.clientX + window.pageXOffset,
        y: event.clientY + window.pageYOffset,
        id: me.id,
        c: me.country // TODO: have server re-look-up country
      });
    };

    on('motion', function(other) {
      // toss junk motion
      if (!(other.x && other.y)) return;

      if (!others[other.id] || !others[other.id].flag)
        showOther(other);

      moveFlag(others[other.id].flag, other.x, other.y);
    });

    var moveFlag = function(flag, x, y) {
      // check x and y to prevent motion on 0,0
      if (x && y) {
        flag.style.left = "" + x + "px";
        flag.style.top = "" + y + "px";
      }
    };

    var mouseWheel = function(event) {
      event = event || window.event;

      // how many clicks, and in which direction
      var direction;
      if (event.wheelDelta)
        direction = (event.wheelDelta > 0) ? 1 : -1;
      else if (event.detail && (event.detail != 0))
        direction = (event.detail > 0) ? -1 : 1;

      var increment = 15;

      if (direction > 0)
        me.angle += increment;
      else
        me.angle -= increment;

      setRotate(me.flag, me.angle);
      emit('scroll', {id: me.id, angle: me.angle});

      if (event.preventDefault) event.preventDefault();
      return false;
    }

    on('scroll', function(other) {
      if (!others[other.id]) return;

      others[other.id].angle = other.angle;
      setRotate(others[other.id].flag, other.angle);
    });

    // Let users use left and right arrow keys to rotate the flag.
    // Especially helpful to users with no scroll wheel! (Who has
    // those things now anyway?)
    //
    // Thanks to Brittany Rose for the idea!
    var keyDown = function(event) {
        event = event || window.event;

        var direction;

        if (event.keyCode == '37') // left
          direction = -1;
        else if (event.keyCode == '39') // right
          direction = 1;
        else
          return true;

        log.debug("key press: " + event.keyCode);

        var increment = 15;

        if (direction > 0)
          me.angle += increment;
        else
          me.angle -= increment;

        setRotate(me.flag, me.angle);
        emit('scroll', {id: me.id, angle: me.angle});

        if (event.preventDefault) event.preventDefault();
        return false;
    }

    var mouseClick = function(event) {
      event = event || window.event;

      var button = clickMap[event.button];

      var x = event.clientX + window.pageXOffset;
      var y = event.clientY + window.pageYOffset;

      clickbacks[button](me, x, y);

      // is quick-rebroadcast, no server processing
      emit('click', {x: x, y: y, id: me.id, button: button});
    };

    on('click', function(other) {
      clickbacks[other.button](others[other.id], other.x, other.y);
    });

    clickbacks.left = function(person, x, y) {
      if (!visible) return;
      if (!person) return;

      var elem = document.createElement("div");

      var width = flagWidth(person.country)
      var height = 20; // constant flag height
      var xOff = Math.floor(width / 2);
      var yOff = Math.floor(height / 2);

      if (canNotify && me.notify && !(Notification.permission == "granted"))
        notifications.permission();

      elem.className = "click";
      elem.style.width = "" + width + "px";
      elem.style.height = "" + height + "px";
      elem.style.left = "" + (x - xOff) + "px";
      elem.style.top = "" + (y - yOff) + "px";

      setRotate(elem, person.angle);

      document.body.appendChild(elem);
      setTimeout(function() {
        elem.style.width = "" + (width*2) + "px";
        elem.style.height = "" + (height*2) + "px";
        elem.style.borderColor = "#fff";
        elem.style.left = "" + (x - width) + "px";
        elem.style.top = "" + (y - height) + "px";
        setTimeout(function() {
          elem.parentElement.removeChild(elem);
        }, 500);
      }, 10);
    };

    // create a translucent ghost flag
    clickbacks.right = function(person, x, y) {
      if (!visible) return;
      if (!person) return;

      // enforce ghost cap
      if (person.ghosts >= user.live.ghost_max)
        return;

      var ghost = flagFor(person.country, "ghost");
      setRotate(ghost, person.angle);
      document.body.appendChild(ghost);
      moveFlag(ghost, x, y);
      person.ghosts += 1;

      // exists for 1s, fades for 1s
      setTimeout(function() {
        ghost.parentElement.removeChild(ghost);
        if (person) person.ghosts -= 1;
      }, user.live.ghost_duration);
    };

    clickbacks.middle = function(person, x, y) {
      person.angle = 0;
      setRotate(person.flag, 0);
    };


/**** Chat mechanics ****/

    // welcome user once, then after every 10 chat messages received.
    // turn off when the user first uses say().
    var said = false;
    var chatsSince = 0;

    var myName = function() {
      var hello = "%cYour name is ";
      hello += "%c" + me.name;
      hello += "%c. Hello, ";
      hello += "%c" + me.name;
      hello += "%c.";
      log.system(hello, styles.system, styles.chat_my_name, styles.system, styles.chat_my_name, styles.system);

      var hasName = false;
      if (window.localStorage) {
        var name = window.localStorage.getItem("me.name", me.name);
        if (name) hasName = true;
      }

      if (!hasName)
        spam();
    }

    var spam = function() {
      var msg = "%cPlease%c don't spam and ruin the chat! :) %c  %c %c  %c %c  ";
      log.system(msg, styles.please, styles.spam, styles.emoji("smiley"), styles.spam, styles.emoji("heart"), styles.spam, styles.emoji("christmas_tree"));
    }

    // I'll have to figure out a better syntax for
    var welcomeUser = function() {
      if (user.live.chat != "true") return;

      log.system("%cCommands:", styles.help);
      log.system(" ");
      var help1a = "%csay%c - Say a message to other people."
      var help1 = "%csay(\"message\")";
      help1 += "%c - Say a %cmessage %cto other people. (Include the quotes and parentheses!)";
      var help2a = "%crename%c - Change to a new name.";
      var help2 = "%crename(\"Name\")";
      help2 += "%c - Change to a new %cName%c. (Include the quotes and parentheses!)";
      var helpX = "%cnotifications%c - Enable desktop notifications for when you are mentioned"
      var help3 = "%chelp%c - See this message again.";
      var help4 = "%cadvanced%c - See menu of advanced options.";
      var help5 = "%crecent%c - See recent chat history.";

      log.system(help1a, styles.help_bold, styles.help);
      log.system(help2a, styles.help_bold, styles.help);
      log.system(help1, styles.help_bold, styles.help, styles.help_bold, styles.help);
      log.system(help2, styles.help_bold, styles.help, styles.help_bold, styles.help);
      log.system(helpX, styles.help_bold, styles.help);
      log.system(help3, styles.help_bold, styles.help);
      log.system(help4, styles.help_bold, styles.help);
      log.system(help5, styles.help_bold, styles.help);
      log.system(" ");

      return " "; // when used via 'help'
    };

    var showAdvanced = function() {
      if (user.live.chat != "true") return;

      var advanced1 = "%ctraffic%c - Turn on/off messages whenever people enter/leave your room. (Currently ";
      advanced1 += "%c" + (user.log.join ? "on" : "off") + "%c.)";
      // var advanced2 = "%cemoji%c - How to use emoji in chat.";
      var advanced3 = "%croom%c - Stats about the room you're in."
      var advanced4 = "%cwtf%c - How is this happening"
      var keys = "%cYou can use arrow keys to rotate the flag."
      var api = "%cWant to make a bot? Check out the JavaScript API: https://github.com/isitchristmas/web/wiki"

      var credit = "%cMade by https://twitter.com/konklone, full credits at https://isitchristmas.com/humans.txt";

      log.system("%cAdvanced commands:", styles.help);
      log.system(" ");
      log.system(advanced1, styles.help_bold, styles.help, styles.help_bold, styles.help);
      // log.system(advanced2, styles.help_bold, styles.help);
      log.system(advanced3, styles.help_bold, styles.help);
      log.system(advanced4, styles.help_bold, styles.help);
      log.system(" ");
      log.system(keys, styles.help);
      log.system(api, styles.help);
      log.system(" ");
      log.system(credit, styles.system);
      spam();

      return " ";
    };

    var showEmoji = function() {
      log.system("%cYou can put :'s around some words to create emoji.", styles.system)
      log.system(" ");
      log.system("%cFor example, %csay(%c\"I am :worried: you will :facepunch: me\"%c)", styles.system, styles.info, styles.info_bold, styles.info);
      log.system("%cwill say: %cI am %c  %c you will %c  %c me", styles.system, styles.info_bold, styles.emoji("worried"), styles.info_bold, styles.emoji("facepunch"), styles.info_bold);
      log.system(" ");
      log.system("%cfull emoji reference (and my data source): http://www.emoji-cheat-sheet.com", styles.system)
      return " ";
    };

    var showRoom = function() {
      // re-key others by country
      var countries = {};
      for (var id in others) {
        var country = others[id].country;
        if (!countries[country]) countries[country] = [];
        countries[country].push(id);
      }

      // sort by size
      var stats = Object.keys(countries);
      stats.sort(function(a, b) {
        return countries[a].length < countries[b].length
      });

      log.system("%cYou're in a randomly chosen 'room' with the other flags. Stats about your room:", styles.system)
      log.system(" ");

      stats.forEach(function(country) {
        var name = Christmas.countries[country].name;
        var num = countries[country].length;
        var people = num == 1 ? "person" : "people";
        log.system("%c[" + country + "]%c[" + name + "] " + num + " " + people, styles.debug, styles.info);
      });

      log.system(" ");
      log.system("%c(Not all connected people are visible.)", styles.system)
      log.system("%c(Chat is universal and goes through every room.)", styles.system);

      return " ";
    };

    var showWtf = function() {
      log.system(" ");

      log.system("%cWhat basically is happening: https://konklone.com/post/isitchristmas-dot-com-2013-more-and-better", styles.system);
      log.system("%cWhat precisely is happening: https://github.com/isitchristmas", styles.system);
      log.system("%cHow this console works: https://konklone.com/post/how-to-hack-the-developer-console-to-be-needlessly-interactive", styles.system);
      log.system("%cData from Christmases past: https://github.com/isitchristmas/data", styles.system);
      log.system(" ");
      log.system("%cSend any fun screenshots/screencasts to %ceric@konklone.com%c.", styles.system, styles.system_bold, styles.system);

      return " ";
    }

    var say = function(message) {
      if (!message) return;
      if (user.live.chat != "true") return;
      message = message.toString();

      said = true;
      emit('chat', {message: message});
      return blank();
    };
    // run without parens to use a popup dialog
    say.toString = function() {
      say(prompt("What do you want to say?"));
      return " ";
    };
    // convenience - handle all caps typos, and s
    var s = S = Say = sAy = saY = SAy = SaY = sAY = SAY = say;

    var rename = function(name) {
      if (!name) return;
      emit('rename', {name: name})
      return blank();
    };

    var notifications = function(to) {
      if (!canNotify) {
        log.system("Unfortunately, your browser doesn't support HTML5 notifications.");
        return;
      }

      if (to === undefined)
        me.notify = !me.notify;
      else
        me.notify = to;

      if (window.localStorage)
        window.localStorage.setItem("me.notify", me.notify);

      if (me.notify) {
        if (Notification.permission == "granted")
          log.system("%cNotifications have been %cenabled%c.", styles.system, styles.system_bold, styles.system);
        else
          log.system("%cNotifications have been %cenabled%c, but you first must %cclick your flag above%c, to prompt your browser to ask permission.", styles.system, styles.system_bold, styles.system, styles.system_bold, styles.system);
      } else
        log.system("%cNotifications have been %cdisabled.", styles.system, styles.system_bold);
    };

    notifications.toString = function() {
      notifications();
      return " ";
    };

    notifications.permission = function() {
      Notification.requestPermission(function(permission) {
        if (permission == "granted") {
          // only useful in Chrome, FF sets this automatically
          Notification.permission = "granted";

          notifications(true);
        } else if (permission == "denied") {
          log.system("You've denied permission for notifications. To change this, you'll have to go into your browser's settings.");
          notifications(false);
        } // other option: 'default', unhandled
      });
    };

    // run without parens to use a popup dialog
    rename.toString = function() {
      rename(prompt("Change your name to:"));
      return " ";
    };

    on('rename', function(data) {
      me.name = data.name;

      if (window.localStorage)
        window.localStorage.setItem("me.name", me.name);

      myName();
    });

    var notify = function(user, message) {
      if (!canNotify) return;

      if (Notification.permission == "granted")
        new Notification(user, {body: message, icon: "https://isitchristmas.com/favicon.ico"});
      else
        log.system('Please enable notifications by clicking your flag above.');
    };


    // recent chat messages
    on('recent', function(data) {
      log.system("Recent chat history:");
      for (var i=0; i<data.chats.length; i++)
        displayChat(data.chats[i]);
    });

    on('chat', function(data) {
      // if user's never chatted, re-welcome them every 10 chat messages
      if (!said) {
        chatsSince += 1;
        if (chatsSince >= 10) {
          log.system(" ");
          welcomeUser();
          myName();
          chatsSince = 0;
        }
      }

      displayChat(data);
    });

    var displayChat = function(data) {

      // Below symbol replacement disabled, since emoji support looks
      // broken in Chrome as of sometime in 2015. :(

      // hack some extra recognized symbols in there
      var symbols = {
        "<3": ":heart:",
        ":\\)": ":smiley_cat:",
        "o_o": ":cat:",
        ";\\)": ":wink:",
        ":\\*": ":kissing_closed_eyes:",
        "!\\?": ":interrobang:",
        "\\^_\\^": ":blush:",
        "</3": ":broken_heart:",
        "\\(c\\)": ":copyright:",
        ":\\(": ":crying_cat_face:",
        ";\\(": ":crying_cat_face:",
        ";_;": ":crying_cat_face:",
        "\\${2,}": ":moneybag:",
        "\\btree\\b": ":christmas_tree:",
      };

      for (var symbol in symbols)
        data.message = data.message.replace(new RegExp(symbol, "gi"), symbols[symbol]);


      // notify if someone else says my name and I've opted in
      if (me.notify && !(data.id == me.id) && data.message.replace(new RegExp(me.name, "gi"), '') != data.message)
        notify(data.name, data.message);

      // /me command?
      var slashMe = (data.message.indexOf("/me ") == 0);
      if (slashMe)
        data.message = data.message.replace(new RegExp("\/me "), '');

      // avoid using logging system here, to do complex coloring
      var message = "";
      var nameStyle;

      // no background image support :(
      if (!backgroundImages)
        message += "%c[" + data.country + "]";
      else
        message += "%c   ";

      if (slashMe) {
        message += "%c * " + data.name;
        nameStyle = styles.chat_message;
      } else {
        message += "%c " + data.name + ":";
        nameStyle = (data.id == me.id ? styles.chat_my_name : styles.chat_name);
      }


      var emojis = [];
      if (backgroundImages) {
        while (found = Emoji.regexp.exec(data.message)) {
          emojis.push(styles.emoji(found[1]));
          emojis.push(styles.chat_message)
        };
        if (emojis.length > 0)
          data.message = data.message.replace(Emoji.regexp, "%c  %c");
      }

      message += "%c " + data.message;

      log.chat.apply(log, [
        message,
        styles.flag(data.country),
        nameStyle,
        styles.chat_message
      ].concat(emojis));
    };



/**** Admin powers ***/

    on('config', function(data) {
      log.info("= live config change: " + data.key + " [" + user.live[data.key] + " -> " + data.value + "]");
      user.live[data.key] = data.value;
    });

    on('command', function(data) {
      log.info("= command: " + data.command + " (" + data.arguments.join(",") + ")");
      (commands[data.command] || noop).apply(null, data.arguments);
    })

    command('blast', function(message) {
      log.system("= Server message: " + message);
    })

    command('reconnect', function() {
      log.system("= Server asked us to reconnect, closing connection");
      socket.close();
    });

    command('refresh', function() {
      log.system("= Server asked us to refresh, whooaaaaa");
      window.location = window.location;
    })


/**** rate limiting utilities **/

    // basic rate-limiter for a static value
    function ratelimit(fn, interval) {
      var last = (new Date()).getTime();
      return (function() {
        var now = (new Date()).getTime();
        if ((now - last) > interval) {
          last = now;
          return fn.apply(null, arguments);
        }
      });
    }

    // special version of ratelimit that looks to live-changeable value
    function ratelimitLive(fn, live) {
      var last = (new Date()).getTime();
      return (function() {
        var now = (new Date()).getTime();
        if ((now - last) > user.live[live]) {
          last = now;
          return fn.apply(null, arguments);
        }
      });
    }


/** christmas midnight effect **/

    // live flip when it's Xmas, and when it's not again
    var flagRule = addCSSRule(".flag");
    var clickRule = addCSSRule("div.click");
    var legendRule = addCSSRule("div#legend");
    var linksRule = addCSSRule("div#links");
    function flagFade(centerTime) {
      // fade out, then in
      setTimeout(function() {
        flagRule.style.opacity = 0;
        legendRule.style.display = "none";
        linksRule.style.display = "none";
        clickRule.style.display = "none";

        setTimeout(function() {
          flagRule.style.opacity = 1;
          legendRule.style.display = "block";
          linksRule.style.display = "block";
          clickRule.style.display = "block";
        }, 5000); // 2 seconds after center

      }, (centerTime - Date.now()) - 3000); // 3 seconds before center (fade is 1s)
    }


    function setTimers() {
      // the init timer to fade in the links, used at christmas time

      setTimeout(function() {
        document.getElementById("links").style.opacity = 1;
      }, 2000);

      links.onmouseover = function() {
        if (me && me.flag) me.flag.style.visibility = "hidden";
      };

      links.onmouseout = function() {
        if (me && me.flag) me.flag.style.visibility = "visible";
      };

      var link = document.getElementById("console");
      if (consoles[BrowserDetect.browser]) {
        link.style.display = "inline-block";
        link.setAttribute("href", consoles[BrowserDetect.browser]);
      }


      var nextXmas = Christmas.thisYear();
      // var nextXmas = new Date(Date.now() + (5 * 1000)); // 5 seconds from now
      // var nextXmas = new Date(Date.now()); // pretend we showed up a few milliseconds before

      nextXmas = nextXmas.getTime();

      var afterXmas = nextXmas + (24 * 60 * 60 * 1000); // 1 day later
      // var afterXmas = nextXmas + (5 * 1000); // 10 seconds later

      // console.log("Christmas set for: " + new Date(nextXmas));
      // console.log("Christmas ends: " + new Date(afterXmas));

      if (checkedAt < nextXmas) {

        flagFade(nextXmas);

        christmasTimer = setTimeout(function() {
          log.system("%c" + Christmas.yes(country), styles.answer);
          updateChristmas(true);
        }, (nextXmas - Date.now()));
      }

      if (checkedAt < afterXmas) {

        flagFade(afterXmas);

        christmasEndTimer = setTimeout(function() {
          log.system("%c" + Christmas.no(country), styles.answer);
          updateChristmas(false);
        }, (afterXmas - Date.now()));
      }
    }


/**** minimalist logging system **/

    // (this logging system got a lot less minimalist once I
    // waded into using %c for styling)

    // any extra arguments are passed in to console.log,
    // assuming log formatting is enabled.
    var log = function(severity, message) {
      if (!user.log[severity]) return;

      // if in plain-text mode, strip out %c and don't pass on extra args
      if (plainConsole)
        console.log(message.replace(/%c/gi, ''));

      else {

        // if any args beyond severity and message, pass them on
        var args = Array.prototype.slice.call(arguments, [2]);
        if (args.length > 0)
          console.log.apply(console, [message].concat(args));

        // otherwise, assume plain text bare message
        else
          console.log(message);
      }
    };

    // generate little log.debug, log.info functions, etc.
    var severities = ["debug", "info", "join", "system", "life", "chat", "public"];
    log.convenience = function(severity) {
      return function(message) {
        // break here too, to avoid too much computation if not needed
        if (!user.log[severity]) return;

        var args = Array.prototype.slice.call(arguments, [1]);

        // if no style given, default to any per-severity style
        if (args.length == 0 && (styles[severity])) {
          message = "%c" + message;
          args = [styles[severity]];
        }

        log.apply(window, [severity, message].concat(args));
      }
    }
    severities.forEach(function(severity) {
      log[severity] = log.convenience(severity);
    });

    var styles = {
      debug: "color: #999999",
      join: "color: #999999",
      info: "color: #444444",
      info_bold: "color: #444444; font-weight: bold",
      system: "color: #336699",
      system_bold: "color: #336699; font-weight: bold",
      life: "color: #ff0000",
      chat_country: "color: #999999",
      chat_name: "color: #006600",
      chat_my_name: "color: #006600; font-weight: bold",
      chat_message: "color: #444444",
      spam: "color: #336699", // #990000 for urgency
      please: "color: #336699; font-weight: bold",
      help: "color: #630053",
      help_bold: "color: #630053; font-weight: bold",
      // how is this possible
      flag: function(country) {
        return "background-image: url(\"https://isitchristmas.com/countries/" + country + ".png\"); background-size: 100% 13px; border: 1px solid #d6d6d6";
      },
      emoji: function(emoji) {
        return "background-image: url(\"https://isitchristmas.com/emojis/" + emoji + ".png\"); background-size: 100%;";
      },
      answer: "color: black; font-weight: bold; font-size: 120pt; text-transform: uppercase; font-family: Arial, sans-serif;"
    }


/**** misc and browser-specific fixes **/

    // if someone types just 'help', display help text.
    // side note: do people know you can do this??
    var help = function() {};
    help.toString = welcomeUser;

    var traffic = function() {};
    traffic.toString = function() {
      user.log.join = !user.log.join;
      log.system("%cTurned traffic messages %c" + (user.log.join ? "on" : "off") + "%c.", styles.system, styles.system_bold, styles.system);
      return " ";
    }

    var advanced = function() {};
    advanced.toString = showAdvanced;

    var emoji = function() {};
    emoji.toString = showEmoji;

    var room = function() {};
    room.toString = showRoom;

    var wtf = function() {};
    wtf.toString = showWtf;

    var recent = function() {};
    recent.toString = function() {
      emit('recent');
      return " ";
    };

    // candy
    var candy = function() {};
    candy.toString = function() {window.location = "http://candybox2.net/";}

    // convenience for functions so they don't say 'undefined' in the console
    var blank = function() {
      // IE doesn't like this method.
      if (window._ie) return undefined;

      var nothing = function() {};
      nothing.toString = function() {return " "};
      return nothing;
    }

    // IE fix: make console.log be okay
    if (!window.console) window.console = {};
    if (!window.console.log) window.console.log = function() {};

    // prevent click-drags from "grabbing" image
    document.body.onmousedown = function(event) {
      event = event || window.event;
      if (event.preventDefault) event.preventDefault();
      else event.returnValue = false;
    };


/**** GO ****/

    // setTimers();
    // connect(url);

  </script>
</body>
</html>
# Testeo

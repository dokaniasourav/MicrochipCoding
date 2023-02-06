<!--<h1>Welcome to SvelteKit</h1>-->
<!--<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>-->
<script>
  import {scale} from 'svelte/transition'

  const E_MOUSE_EVENT = {
    MOVE: 111,
    PRESS: 222,
    RELEASE: 333
  }

  const C_STATE = {
    PRESSED: 222,
    RELEASED: 333
  }

  class XY {
    constructor(x, y) {
      this._x = x;
      this._y = y;
    }
    get x() { return this._x; }
    get y() { return this._y; }
    set x(xx) { this._x = xx; }
    set y(yy) { this._y = yy; }
    set xy(pos) { this._x = pos.x; this._y = pos.y; }
    sub(pos1, pos2) {
      this._x = pos1.x - pos2.x; this._y = pos1.y - pos2.y;
      return this;
    }
    add(pos1, pos2) {
      this._x = pos1.x + pos2.x; this._y = pos1.y + pos2.y;
      return this;
    }
  }

  class MouseEvent {
    constructor(_type, pos, nb) {
      this.tp = _type;
      this.nb = nb;
      this.pos = pos;
    }
  }

  const width = 150;
  const height = 150;

  let abs_mouse = new XY(0, 0);
  let rel_mouse = new XY(0, 0);
  let cur_mouse = new XY(0, 0);
  let box_final = new XY(0, 0);
  let box_curr  = new XY(0, 0);
  let box_state = C_STATE.RELEASED;

  function set_all (mouse_event) {
    if (mouse_event.tp === E_MOUSE_EVENT.PRESS) {
      abs_mouse.xy = mouse_event.pos;
      if(abs_mouse.x < box_final.x || abs_mouse.y < box_final.y) {
        return;
      }
      if(abs_mouse.x > (box_final.x + width) || abs_mouse.y > (box_final.y + height)) {
        return;
      }
      box_state = C_STATE.PRESSED;
      rel_mouse.xy = rel_mouse.sub(mouse_event.pos, box_final);
    } else if (mouse_event.tp === E_MOUSE_EVENT.RELEASE) {
      box_state = C_STATE.RELEASED;
      box_final.xy = box_curr;
    } else if (mouse_event.tp === E_MOUSE_EVENT.MOVE) {
      cur_mouse.xy = mouse_event.pos;
      if (box_state === C_STATE.PRESSED) {
        if (mouse_event.nb !== 0) {
          set_box_position(mouse_event);
        } else {
          box_state = C_STATE.RELEASED;
          box_final.xy = box_curr;
          // console.log('Box final corner case = ', box_final);
        }
      }
    }
  }

  function set_box_position(mouse_event) {
    mouse_event.pos.sub(mouse_event.pos, abs_mouse);
    box_curr.xy = box_curr.add(box_final, mouse_event.pos);

    if (box_curr.x < 0) {
      box_curr.x = 0;
    } else if (box_curr.x > (window.innerWidth - width)) {
      box_curr.x = window.innerWidth - width;
    }

    if (box_curr.y < 0) {
      box_curr.y = 0;
    } else if (box_curr.y > (window.innerHeight - height)) {
      box_curr.y = window.innerHeight - height;
    }
  }

  let visibleCircle = false;
  $: {
    visibleCircle = (box_state === C_STATE.PRESSED);
  }

  let circle_style = '';
  $: {
    circle_style = `left: ${rel_mouse.x}px; top: ${rel_mouse.y}px;`;
  }

  let container_style = '';
  const box_dim_style = `width: ${width}px; height: ${height}px;`;
  $: {
    container_style = `${box_dim_style} left: ${box_curr.x}px; top: ${box_curr.y}px;`
  }

  let main_box_style;
  $: {
    let col_f =  box_state === C_STATE.PRESSED ? '#d2d720' : '#625454';
    main_box_style = `background-color: ${col_f};`
  }

  const handleMouseMovement = (e, type) => {
    let pos = new XY(e.clientX, e.clientY);
    let event = new MouseEvent(type, pos, e.buttons);
    set_all(event);
  }

</script>

<div id="super"
     on:mousemove={(e) => handleMouseMovement(e, E_MOUSE_EVENT.MOVE)}
     on:mouseup={(e) => handleMouseMovement(e, E_MOUSE_EVENT.RELEASE)}
     on:mousedown={(e) => handleMouseMovement(e, E_MOUSE_EVENT.PRESS)}
>
  <div id="container" style={container_style}>
    <div id=main-box style={main_box_style}>
      {#if box_state === C_STATE.PRESSED}
        Click at: <br>
        X: {rel_mouse.x} <br>
        Y: {rel_mouse.y}
      {/if}
    </div>
    <div id="title">
      BOX XY = {box_curr.x + ', ' + box_curr.y}
    </div>
    {#if visibleCircle}
      <span id="circle" out:scale="{{duration: 1000, delay: 0.1, opacity: 0, start: 25}}" style="{circle_style}"></span>
    {/if}
    <div class="debug">
      X = {cur_mouse.x}, Y = {cur_mouse.y}
    </div>
  </div>
</div>

<style>
  * {
    user-select: none;
    -webkit-user-drag: none;
  }
  #super{
    display: block;
    height: 100vh;
    margin: 0;
    padding: 0;
  }
  #container {
    text-align: center;
    /*overflow: hidden;*/
    background-color: #facade;
    display: block;
    border: 1px solid #a3ffee;
    border-radius: 1px;
    position: fixed;
    top: 0;
    left: 0;
    width: 100px;
    height: 100px;
    transition: background 400ms;
  }
  #main-box {
    width: 100%;
    height: 80%;
    background-color: #5c5f65;
    position: relative;
    top: 0;
  }
  #title {
    width: 100%;
    height: 20%;
    background-color: #1d1f1d;
    color: #facade;
  }
  .debug {
    color: darkgray;
    padding: 0;
    margin: 0;
  }

  #circle {
    padding: 1px;
    position: absolute;
    border-radius: 50%;
    background-color: rgba(255, 255, 255, 1.0);
  }

</style>
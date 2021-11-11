# Fullscreen mode

This guide will give you the code in order to switch between window, borderless and fullscreen modes.

This guide is a follow up to [Get started](./get-started.md).

Here is some code that can toggle your game's fullscreen mode with borderless as an option.

```csharp
bool _isFullscreen = false;
bool _isBorderless = false;
int _width = 0;
int _height = 0;
GraphicsDeviceManager _graphics;
GameWindow _window;

public void ToggleFullscreen() {
    bool oldIsFullscreen = _isFullscreen;

    if (_isBorderless) {
        _isBorderless = false;
    } else {
        _isFullscreen = !_isFullscreen;
    }

    ApplyFullscreenChange(oldIsFullscreen);
}
public void ToggleBorderless() {
    bool oldIsFullscreen = _isFullscreen;

    _isBorderless = !_isBorderless;
    _isFullscreen = _isBorderless;

    ApplyFullscreenChange(oldIsFullscreen);
}

private void ApplyFullscreenChange(bool oldIsFullscreen) {
    if (_isFullscreen) {
        if (oldIsFullscreen) {
            ApplyHardwareMode();
        } else {
            SetFullscreen();
        }
    } else {
        UnsetFullscreen();
    }
}
private void ApplyHardwareMode() {
    _graphics.HardwareModeSwitch = !_isBorderless;
    _graphics.ApplyChanges();
}
private void SetFullscreen() {
    _width = Window.ClientBounds.Width;
    _height = Window.ClientBounds.Height;

    _graphics.PreferredBackBufferWidth = GraphicsAdapter.DefaultAdapter.CurrentDisplayMode.Width;
    _graphics.PreferredBackBufferHeight = GraphicsAdapter.DefaultAdapter.CurrentDisplayMode.Height;
    _graphics.HardwareModeSwitch = !_isBorderless;

    _graphics.IsFullScreen = true;
    _graphics.ApplyChanges();
}
private void UnsetFullscreen() {
    _graphics.PreferredBackBufferWidth = _width;
    _graphics.PreferredBackBufferHeight = _height;
    _graphics.IsFullScreen = false;
    _graphics.ApplyChanges();
}
```

To use this, call either `ToggleFullscreen` or `ToggleBorderless`.

## Explanation

Track the current fullscreen state. Initialized to false since the game starts in window mode.

```csharp
bool _isFullscreen = false;
bool _isBorderless = false;
```

Window mode happens when both `_isFullscreen` and `_isBorderless` are false.

Borderless mode happens when both `_isFullscreen` and `_isBorderless` are true.

Fullscreen mode happens when `_isFullscreen` is true and `_isBorderless` is false.

---

Going fullscreen means changing the current window's width and height. It's a good idea to save the previous values so that we can restore the window mode. Initialized to 0 but the actual value will be set by SetFullscreen.

```csharp
int _width = 0;
int _height = 0;
```

---

Borderless mode happens when the hardware mode switch is set to false. This method is called when the game is already in fullscreen.

```csharp
private void ApplyHardwareMode() {
    _graphics.HardwareModeSwitch = !_isBorderless;
    _graphics.ApplyChanges();
}
```

---

Going fullscreen we preserve the current window size. Then we apply the size of the monitor that the window is currently in. This method handles the current borderless state.

```csharp
private void SetFullscreen() {
    _width = Window.ClientBounds.Width;
    _height = Window.ClientBounds.Height;

    _graphics.PreferredBackBufferWidth = GraphicsAdapter.DefaultAdapter.CurrentDisplayMode.Width;
    _graphics.PreferredBackBufferHeight = GraphicsAdapter.DefaultAdapter.CurrentDisplayMode.Height;
    _graphics.HardwareModeSwitch = !_isBorderless;

    _graphics.IsFullScreen = true;
    _graphics.ApplyChanges();
}
```

---

To go back to the window mode, we restore the previously saved sizes.

```csharp
private void UnsetFullscreen() {
    _graphics.PreferredBackBufferWidth = _width;
    _graphics.PreferredBackBufferHeight = _height;
    _graphics.IsFullScreen = false;
    _graphics.ApplyChanges();
}
```

---

There are three modes in total: window, borderless, fullscreen. To switch between borderless and fullscreen, it's just a matter of calling `ApplyHardwareMode`. To switch between window and the other two, it's a matter of calling `SetFullscreen` or `UnsetFullscreen`.

```csharp
private void ApplyFullscreenChange(bool oldIsFullscreen) {
    if (_isFullscreen) {
        if (oldIsFullscreen) {
            ApplyHardwareMode();
        } else {
            SetFullscreen();
        }
    } else {
        UnsetFullscreen();
    }
}
```

---

Toggling the fullscreen mode is a matter of turning off the borderless mode or toggling the fullscreen state.

```csharp
public void ToggleFullscreen() {
    bool oldIsFullscreen = _isFullscreen;

    if (_isBorderless) {
        _isBorderless = false;
    } else {
        _isFullscreen = !_isFullscreen;
    }

    ApplyFullscreenChange(oldIsFullscreen);
}
```

---

Toggling the borderless mode is a matter of toggling the borderless state and making the fullscreen state be the same.

```csharp
public void ToggleBorderless() {
    bool oldIsFullscreen = _isFullscreen;

    _isBorderless = !_isBorderless;
    _isFullscreen = _isBorderless;

    ApplyFullscreenChange(oldIsFullscreen);
}
```

## Follow up

[Persistent game settings](../tutorial/game-settings/README.md), a follow up tutorial that applies this fullscreen mechanism.

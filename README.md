# Learn MonoGame

Documentation to learn MonoGame from scratch.

To get started, read the [Get started](https://learn-monogame.github.io/how-to/get-started/) guide.

## Discussions

Feel free to leave feedback or request subjects to cover in the [Discussions](https://github.com/learn-monogame/learn-monogame.github.io/discussions) board.

## What is MonoGame?

[MonoGame](https://www.monogame.net/) is an open-source framework, a thin layer of abstraction over input, sound, and graphics APIs. MonoGame lets game developers write cross platform code that will run on desktop, mobile, and console devices. Many commercially successful indie games have been shipped using MonoGame, and it's similar frameworks XNA and [FNA](https://fna-xna.github.io/), since 2007. MonoGame is ideal for developers who don't want an engine to dictate their decisions and rather have more control in the development of their game. It has a lot of features for 2D already built-in, but doesn't have the type of features that Unreal or Unity have for 3D out of the box. MonoGame tries to get out of the developer's way by providing only the essentials: input, sound, and drawing to the screen.

MonoGame supports Windows, Mac, Linux, Android, iOS, Windows Phone, Nintendo Switch, PlayStation 4, PsVita, PlayStation Mobile, Microsoft Xbox 360, Xbox One, Google Stadia, Raspberry Pi, and more in the future. MonoGame can support all these platforms thanks to the work done by the MonoGame Team and support from the community. MonoGame is built around C#, which means the executable program is compiled to Common Intermediate Language (CIL). However, [Sickhead Games](https://www.sickheadgames.com/) created a tool called [Brute](http://brute.rocks/) that converts CIL into C++. (The code is not shared publicly due to some issues with NDAs for consoles.) This is comparable to what Unity does with [IL2CPP](https://docs.unity3d.com/Manual/IL2CPP.html). Combining the transpiled and compiled C++ with the proper libraries allows the game executable to run on many different platforms with great performance. Thanks to the nature of Brute and it's design, this particular workflow allows older games to be ported to newer systems ([one example](https://twitter.com/sickhead/status/730783172602929152) is that Sickhead Games helped port Stardew Valley to Xbox One, PlayStation 4, and Nintendo Switch), and is seen as a good framework to develop in for this exact reason. Porting from a framework or engine that is closed source is often difficult, time consuming, expensive, or impossible. It's much safer to work with an open-source framework that you can modify as needed to suit your particular set of requirements. MonoGame and FNA provide this security, while also being mature, stable, and established frameworks.

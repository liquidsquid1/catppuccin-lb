<h3 align="center">
	<img src="https://raw.githubusercontent.com/catppuccin/catppuccin/main/assets/logos/exports/1544x1544_circle.png" width="100" alt="Logo"/><br/>
	<img src="https://raw.githubusercontent.com/catppuccin/catppuccin/main/assets/misc/transparent.png" height="30" width="0px"/>
	Catppuccin for <a href="https://liquidbounce.net/">LiquidBounce</a>
	<img src="https://raw.githubusercontent.com/catppuccin/catppuccin/main/assets/misc/transparent.png" height="30" width="0px"/>
</h3>

<p align="center">
    <a href="https://github.com/liquidsquid1/catppuccin-lb/stargazers"><img src="https://img.shields.io/github/stars/liquidsquid1/catppuccin-lb?colorA=363a4f&colorB=b7bdf8&style=for-the-badge"></a>
    <a href="https://github.com/liquidsquid1/catppuccin-lb/issues"><img src="https://img.shields.io/github/issues/liquidsquid1/catppuccin-lb?colorA=363a4f&colorB=f5a97f&style=for-the-badge"></a>
    <a href="https://github.com/liquidsquid1/catppuccin-lb/contributors"><img src="https://img.shields.io/github/contributors/liquidsquid1/catppuccin-lb?colorA=363a4f&colorB=a6da95&style=for-the-badge"></a>
</p>

## Usage

1. Download your preferred flavour
> [!NOTE]  
> I am only currently maintaining Mocha (Mauve), however you may change the colour in colors.scss and rebuild!

```
git clone htpps://github.com/liquidsquid1/catppuccin-lb.git
```

2. Compile it using Node
> [!NOTE]
> This assumes you already have Node installed.

```
cd catppuccin-lb
npm install
npm run build
```

3. Move it into your client folder
Navigate to catppuccin-lb/dist and place the dist folder in your themes folder
> [!NOTE]
> Don't know how to get to your themes folder? Simply type `.client theme browse` in your minecraft chat and it will open up the directory.

4. Set your theme
> [!WARNING]
> This will override your current hud layout, including your positions of elements.

Simply type `.client theme set dist` in your minecraft chat.

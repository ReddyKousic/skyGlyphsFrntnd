# 🌍 SkyGlyphs Frontend

Generate your name using real satellite imagery from NASA's Landsat program! SkyGlyphs creates beautiful words and names by combining authentic satellite images that naturally resemble letters.

![SkyGlyphs Demo](https://github.com/ReddyKousic/skyGlyphsFrntnd/blob/main/static/skyglyphs-landsat-min.png?raw=true)

## ✨ Features

- **Real Satellite Imagery**: Uses authentic NASA Landsat satellite images, not AI-generated content
- **Interactive Generation**: Create variations of your name with different satellite images
- **Download Capability**: Save your generated images as PNG files
- **Location Details**: See exactly where each letter image was captured
- **Responsive Design**: Works seamlessly on desktop and mobile devices
- **Community-Driven**: Built on contributions from satellite imagery enthusiasts

## 🚀 Live Demo

Visit [skyglyphs.koastec.com](https://skyglyphs.koastec.com) to try SkyGlyphs now!

## 🛠️ Tech Stack

- **Framework**: [SvelteKit](https://kit.svelte.dev/) with Svelte 5
- **Styling**: [Tailwind CSS](https://tailwindcss.com/)
- **Language**: TypeScript
- **Image Processing**: HTML5 Canvas API
- **Hosting**: Vercel
- **Image CDN**: Statically.io via GitHub repository

## 🏃‍♂️ Quick Start

### Prerequisites

- Node.js 18+ 
- pnpm

### Installation

```bash
# Clone the repository
git clone https://github.com/ReddyKousic/skyGlyphsFrntnd.git
cd skyGlyphsFrntnd

# Install dependencies
pnpm install

# Start development server
pnpm dev
```

Visit `http://localhost:5173` to see the app running locally.

## 🎯 How It Works

1. **Input**: User enters a word or name
2. **Image Selection**: For each letter, a random satellite image is selected from the manifest
3. **Canvas Rendering**: Images are combined into a single word using HTML5 Canvas
4. **Display**: The generated image is shown with location details for each letter
5. **Download**: Users can save the final image as a PNG file

## 🌟 Key Components

### Main Features

- **Word Generation**: Combines individual letter images into complete words
- **Random Variation**: Each generation uses different satellite images for variety
- **Location Parsing**: Extracts place and country information from image filenames
- **Download Functionality**: Converts canvas to downloadable PNG with loading states

### Image Processing

- Images are scaled to 75% of original size for optimal display
- Cross-origin requests are handled for external CDN images
- Canvas operations combine multiple images with proper positioning

## 🔗 Related Repositories

- [Satellite Letters Collection](https://github.com/ReddyKousic/sky_glyphs_satellite_letters_webp) - The image dataset used by SkyGlyphs

## 🤝 Contributing

We welcome contributions! Here are ways you can help:

### Frontend Contributions

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Commit your changes (`git commit -m 'Add amazing feature'`)
5. Push to the branch (`git push origin feature/amazing-feature`)
6. Open a Pull Request

### Satellite Image Contributions

Help expand our letter collection by contributing satellite images to the [main dataset repository](https://github.com/ReddyKousic/sky_glyphs_satellite_letters_webp).

## 📝 Development Notes

### Environment Setup

The app fetches satellite images from:
```
https://cdn.statically.io/gh/ReddyKousic/sky_glyphs_satellite_letters_webp/main
```

### Image Format Requirements

- Images should be in WebP format
- Filename format: `{letter}-{place}-{country}-{optional-number}.webp`
- Example: `a-newyork-usa-1.webp`

### Manifest Structure

The `manifest.json` file organizes images by letter:
```json
{
  "a": ["a-location1-country1.webp", "a-location2-country2.webp"],
  "b": ["b-location1-country1.webp"],
  ...
}
```

## 🐛 Bug Reports

Found a bug? Please open an issue with:

- Clear description of the problem
- Steps to reproduce
- Expected vs actual behavior
- Browser and device information
- Screenshots if applicable

## 💡 Feature Requests

Have an idea for SkyGlyphs? Open an issue with:

- Clear description of the feature
- Use case and benefits
- Mockups or examples (if applicable)

## 🙏 Acknowledgments

- **Inspiration**: NASA's ["Your Name in Landsat"](https://science.nasa.gov/specials/your-name-in-landsat/)
- **Original Concept**: Ross Walter, Allison Nussbaum & Ginger Butcher (NASA Landsat Team)
- **Satellite Data**: NASA's [Landsat Program](https://landsat.gsfc.nasa.gov/)
- **Community**: All the contributors who help expand our satellite letter collection

## 📞 Contact

**Kousic Thavva** - Project Creator & Maintainer

- GitHub: `@ReddyKousic`
- Twitter: `@thkousic`

---
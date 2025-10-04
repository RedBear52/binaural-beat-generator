# Binaural Beat Generator

A Vue.js application for generating programmable binaural beats with advanced audio features. This app allows users to create customized binaural beats for meditation, focus, relaxation, and other therapeutic purposes.

## Features

### Core Functionality

- **Dual Frequency Control**: Set independent frequencies for left and right ear channels
- **Real-time Beat Calculation**: Displays resulting binaural beat frequency and type
- **Audio Player Controls**: Full playback control with volume adjustment
- **Background Noise**: Optional white noise and pink noise generation
- **Beat Type Classification**: Automatically categorizes beats (Delta, Theta, Alpha, Beta, Gamma)

### Audio Features

- **Web Audio API Integration**: High-quality audio synthesis
- **Stereo Processing**: True binaural audio with proper channel separation
- **Noise Generation**: Mathematically accurate white and pink noise algorithms
- **Real-time Updates**: Frequency changes apply immediately during playback

### User Interface

- **Responsive Design**: Works on desktop and mobile devices
- **Intuitive Controls**: Easy-to-use sliders and input fields
- **Visual Feedback**: Real-time display of frequency data and playback status
- **Modern Design**: Clean, professional interface

## Getting Started

### Prerequisites

- Node.js 18.17.1 or higher
- npm or yarn package manager

### Installation

1. Clone the repository:

```bash
git clone [repository-url]
cd co-pilot-guided-prototype
```

2. Install dependencies:

```bash
npm install
```

3. Start the development server:

```bash
npm run dev
```

4. Open your browser and visit `http://localhost:5173/` (or the port shown in terminal)

### Build for Production

```bash
npm run build
```

### Preview Production Build

```bash
npm run preview
```

## Usage

### Basic Operation

1. **Set Frequencies**:

   - Use the number inputs to set the left and right ear frequencies
   - Range: 20-2000 Hz with 0.1 Hz precision
   - The app automatically calculates the binaural beat frequency

2. **Play Audio**:

   - Click the Play button to start audio generation
   - Use the volume slider to adjust playback level
   - Click Pause to stop playback

3. **Add Background Noise**:
   - Toggle white noise or pink noise using the checkboxes
   - Adjust noise volume with the individual sliders
   - Noise can be toggled on/off during playback

### Understanding Binaural Beats

The app displays the beat type based on the frequency difference:

- **Delta (0.5-4 Hz)**: Deep sleep, healing
- **Theta (4-8 Hz)**: Meditation, creativity
- **Alpha (8-13 Hz)**: Relaxation, focus
- **Beta (13-30 Hz)**: Alertness, concentration
- **Gamma (30+ Hz)**: High-level cognitive functioning

### Tips for Use

- **Start with Low Volume**: Begin with 30-50% volume and adjust as needed
- **Use Headphones**: Binaural beats require stereo separation
- **Session Length**: Start with 10-15 minute sessions
- **Comfortable Frequencies**: Most effective range is 100-1000 Hz for carrier frequencies

## Technical Details

### Architecture

- **Vue 3**: Modern composition API with `<script setup>`
- **Vite**: Fast build tool and development server
- **Web Audio API**: Native browser audio processing
- **No External Dependencies**: Pure JavaScript audio synthesis

### Audio Implementation

- **Oscillators**: Sine wave generators for each ear
- **Channel Merger**: Proper stereo separation
- **Gain Nodes**: Volume control and mixing
- **Buffer Sources**: Noise generation with mathematical algorithms

### Browser Compatibility

- Chrome/Edge 66+
- Firefox 60+
- Safari 14.1+
- Requires HTTPS for some features in production

## Future Enhancements

### Planned Features

- **Programmable Sessions**: Preset programs with time-based frequency changes
- **Audio Visualization**: Real-time visual patterns that react to the audio
- **External Audio Support**: Mix with other apps (meditation tracks, music)
- **Preset Library**: Common binaural beat programs for different purposes
- **Session Recording**: Save and replay custom sessions
- **Brainwave Monitoring**: Integration with EEG devices

### Optional Advanced Features

- **Multi-app Audio**: Background compatibility with other audio apps
- **Visual Patterns**: Audio-reactive geometric visualizations
- **Frequency Sweeps**: Gradual frequency transitions
- **Binaural Beat Sequences**: Complex multi-stage programs

## Development

### Project Structure

```
src/
├── App.vue                           # Main application component
├── main.js                           # Application entry point
├── style.css                         # Global styles
└── components/
    └── BinauralBeatGenerator.vue     # Main binaural beat component
```

### Key Technologies

- Vue 3 Composition API
- Web Audio API
- CSS Grid/Flexbox
- ES6+ JavaScript

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## License

This project is licensed under the MIT License.

## Support

For questions or issues, please create an issue in the GitHub repository.

## Disclaimer

This application is for educational and wellness purposes. It is not intended to diagnose, treat, cure, or prevent any medical condition. Always consult healthcare professionals for medical advice.

Learn more about IDE Support for Vue in the [Vue Docs Scaling up Guide](https://vuejs.org/guide/scaling-up/tooling.html#ide-support).

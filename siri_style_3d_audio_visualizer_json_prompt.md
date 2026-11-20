{
  "project_metadata": {
    "title": "AETHER // ACOUSTIC CORE - Siri-Style Interactive 3D Audio Visualizer",
    "role": "Expert creative front-end developer specializing in Three.js, WebGL, GLSL shaders, and Web Audio API",
    "architecture": "Single-file standalone HTML (index.html) with inline CSS and modular ES6+ classes",
    "objective": "Create a sleek, dark, minimalist interactive 3D audio visualizer orb that reacts not only to raw frequency levels (bass, mid, treble) but specifically to dynamic voice volume changes, sudden vocal attacks, and syllable transients."
  },
  "dependencies_and_cdn": {
    "styling": "https://cdn.tailwindcss.com",
    "font": {
      "family": "Inter",
      "weights": [300, 400, 500, 600],
      "url": "https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600&display=swap"
    },
    "three_js": {
      "version": "r128",
      "core": "https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js",
      "post_processing": [
        "https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/postprocessing/EffectComposer.js",
        "https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/postprocessing/RenderPass.js",
        "https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/postprocessing/ShaderPass.js",
        "https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/shaders/CopyShader.js",
        "https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/shaders/LuminosityHighPassShader.js",
        "https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/postprocessing/UnrealBloomPass.js"
      ]
    }
  },
  "visual_design_and_ui": {
    "theme": {
      "background_color": "#050508",
      "text_primary": "#f8fafc",
      "palette": {
        "deep_core": "#080a1a",
        "electric_cyan": "#0dd9ff",
        "electric_blue": "#2673ff",
        "neon_purple": "#ad38fa"
      }
    },
    "hud_layout": {
      "container": "Fullscreen pointer-events-none flex column overlay over fixed WebGL canvas (z-index: 1)",
      "header": {
        "brand_badge": "Circular gradient ring (cyan-400 -> blue-500 -> purple-500) with pulsing cyan center dot, title 'AETHER // ACOUSTIC CORE', subtitle 'Interactive 3D Neural Audio Orb'",
        "telemetry_pill": "Glassmorphism pill showing live status dot (with CSS @keyframes pulse-ring), status label ('Standby' / 'Live Mic' / 'Synth Wave'), and 3 vertical mini frequency bars (#bar-bass, #bar-mid, #bar-treble)"
      },
      "notification_toast": "Centered non-intrusive glassmorphism toast banner (#toast-message) with smooth opacity fade (4.2s timeout) instead of browser alert()",
      "footer_controls": {
        "primary_button": "'Start Audio / Enable Mic' glassmorphism pill button with microphone SVG icon and hover glow shadow",
        "secondary_button": "'Switch to Synth Demo' / 'Retry Microphone' pill button (revealed once audio starts)",
        "hint_text": "'Move cursor or drag to tilt orb • Reacts to real-time acoustic frequencies'"
      }
    }
  },
  "phase_1_audio_engine": {
    "class_name": "AudioEngine",
    "web_audio_config": {
      "fftSize": 512,
      "smoothingTimeConstant": 0.82,
      "microphone_constraints": {
        "echoCancellation": true,
        "noiseSuppression": true,
        "autoGainControl": false
      },
      "fallback_behavior": "Gracefully catch getUserMedia denial/unavailability and switch to a procedural multi-harmonic synthetic wave simulator mimicking speech cadences and beat pulses."
    },
    "frequency_bands_and_normalization": {
      "bass": "Bins 1..10 (~20Hz-250Hz), normalized 0..1 with pow(0.85) perceptual curve",
      "mid": "Bins 11..64 (~250Hz-2500Hz), normalized 0..1 with pow(0.85) perceptual curve",
      "treble": "Bins 65..180 (~2500Hz-14000Hz), normalized 0..1 with pow(0.8) perceptual curve",
      "smoothing": "Frame-rate independent exponential lerp: 1.0 - Math.exp(-14.0 * Math.min(deltaTime, 0.1))"
    },
    "voice_volume_change_and_transient_detection": {
      "raw_volume_formula": "bass * 0.4 + mid * 0.45 + treble * 0.15",
      "adaptive_noise_floor": "Tracks ambient background level (fast drop rate 3.5 when quieter than floor, slow rise rate 0.35 when louder) so steady background hum settles out",
      "dynamic_volume": "Math.max(0.0, rawVolume - ambientFloor * 0.75)",
      "volume_delta_calculation": "Math.max(0.0, dynamicVolume - prevVolume)",
      "transient_envelope": "Amplifies positive volume changes by 6.5x (clamped 0..1) using an asymmetric attack/release filter: fast attack (-28.0) on rising volume spikes and smooth decay (-7.5) on release, outputting normalized 'delta' (0.0 to 1.0)"
    }
  },
  "phase_2_glsl_shaders": {
    "object_name": "OrbShaders",
    "uniforms": [
      "uTime",
      "uBass",
      "uMid",
      "uTreble",
      "uVolume",
      "uDelta",
      "uNoiseFreq",
      "uNoiseAmp"
    ],
    "vertex_shader": {
      "noise_algorithm": "Inline 3D Simplex Noise (Ashima Arts / Ian McEwan snoise)",
      "displacement_layers": [
        "Macro liquid wave: snoise(pos * uNoiseFreq + vec3(uTime * 0.55, -uTime * 0.4, uTime * 0.35)) * uNoiseAmp",
        "Mid harmonic fold: snoise(pos * (uNoiseFreq * 2.1) - vec3(uTime * 0.8, uTime * 0.65, -uTime * 0.5)) * (uNoiseAmp * 0.42 * (0.3 + uMid * 1.4))",
        "Treble micro-shimmer: snoise(pos * 5.2 + vec3(uTime * 1.9, uTime * 1.4, -uTime * 1.7)) * (0.045 * uTreble)",
        "Vocal transient ripple wave: sin(pos.y * 8.5 - uTime * 7.5) * cos(pos.x * 6.0 + uTime * 5.0) * (0.085 * uDelta)"
      ],
      "radial_scale": "1.0 + uBass * 0.18 + uDelta * 0.14",
      "normal_recalculation": "Finite-difference tangent/bitangent cross-product perturbation (eps = 0.015) so specular and Fresnel lighting accurately track surface waves"
    },
    "fragment_shader": {
      "iridescence": "Cosine palette driven by vDisplacement, vWorldNormal.xy, uTime, uMid, and uDelta, blended with Electric Blue, Neon Purple, and Electric Cyan",
      "fresnel_rim": "pow(1.0 - max(dot(viewDir, normal), 0.0), 2.6) scaled by (0.75 + uVolume * 1.45 + uBass * 0.5 + uDelta * 1.25)",
      "specular": "Dual directional Blinn-Phong highlights (cyan & purple) modulated by uTreble and uMid",
      "inner_core_pulse": "Adds electricBlue * (1.0 - fresnel) * (uBass * 0.24 + uDelta * 0.35)"
    }
  },
  "phase_3_threejs_scene": {
    "class_name": "VisualizerScene",
    "camera_and_renderer": {
      "camera": "PerspectiveCamera(45, aspect, 0.1, 100), z = 5.2 (or 6.2 on mobile < 640px)",
      "fog": "FogExp2(0x050508, 0.065)",
      "renderer": "WebGLRenderer with antialias: true, ACESFilmicToneMapping (exposure 1.15), pixelRatio capped at 2"
    },
    "meshes": {
      "primary_orb": "IcosahedronGeometry(1.35, 64) with transparent ShaderMaterial",
      "wireframe_halo": "IcosahedronGeometry(1.37, 24) with additive Fresnel wireframe ShaderMaterial",
      "ambient_particles": "220 BufferGeometry points distributed in spherical shell (radius 2.4..6.9) colored cyan/blue/purple with additive blending"
    },
    "post_processing": {
      "composer": "EffectComposer with RenderPass and UnrealBloomPass (strength: 0.65, radius: 0.55, threshold: 0.22)",
      "dynamic_bloom": "Bloom strength modulates in real-time: 0.55 + audioBands.volume * 0.5 + audioBands.delta * 0.55"
    },
    "interactivity": {
      "pointer_tracking": "Mousemove and touchmove tilt targetRotation (x: ±0.45, y: ±0.55) with smooth exponential damping (-5.5) layered over slow continuous Y/Z auto-rotation",
      "responsive_resize": "Updates camera aspect, mobile Z distance, renderer size, pixel ratio, and EffectComposer size"
    }
  },
  "phase_4_animation_loop": {
    "entrypoint": "window.onload",
    "loop_steps": [
      "Compute deltaTime and elapsedTime via THREE.Clock",
      "Update AudioEngine to retrieve smoothed bass, mid, treble, volume, and transient delta",
      "Update HUD frequency bar heights in real time",
      "Pass audio metrics to VisualizerScene.update() to modulate shader uniforms (including uTime speed boost from uDelta), bloom strength, and render frame"
    ]
  }
}
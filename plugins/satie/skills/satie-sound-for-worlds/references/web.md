# Any web page

## Any web page (no bundler)

```html
<script type="module">
  import { SatieEngine } from 'https://www.satie.world/runtime/satie-runtime.js';
  const engine = new SatieEngine();
  const samples = await (await fetch('/satie/samples.json')).json();
  await Promise.all(Object.entries(samples).map(([clip, s]) => engine.loadAudioFile(clip, s.url)));
  engine.loadScript(await (await fetch('/satie/scene.satie')).text());
  document.getElementById('start').onclick = () => engine.play();
  // per frame: engine.setListenerFromMatrix(cameraWorldMatrixColumnMajor) or
  // engine.setListenerPosition(x, y, z) + engine.setListenerOrientation(fx, fy, fz, ux, uy, uz)
  // objects: engine.setHostPosition('fountain', x, y, z)
</script>
```

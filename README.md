# latent-space-explorer

An interactive 3D viewer for feature spaces. Four linked scenes, one self-contained HTML file,
no build step and no server: open `index.html` or use the hosted page.

**Live page:** https://chenhsieh.github.io/latent-space-explorer/

The point of the viewer is that a scatter in a learned space is only readable when the axes
mean something. Every scene here is built on measured axes rather than a generic 2D embedding,
and each scene states what its own axes are.

## Controls

| input | effect |
|---|---|
| drag | rotate |
| scroll or pinch | zoom |
| click a point | show its coordinates in the readout |
| `1`-`4` | switch scene |
| `r` | reset the camera |
| `space` | play the scene move, where a scene has one |

The left rail toggles series on and off, jumps the camera to fixed viewpoints, and holds a
"how to read this frame" note plus the per-scene footnotes.

## The example dataset

The bundle shipped here is embeddings from a codon-sequence classifier that scores whether a
coding sequence looks viral or host in origin, together with sequences that have been rewritten
synonymously. A synonymous rewrite changes the codons but not the protein they encode, so every
rewritten point in the plot has the same amino acid sequence as the parent it came from.

2114 points across four scenes:

| scene | what it shows |
|---|---|
| The decision frame | every sequence on three measured axes at once |
| Frozen, then hardened | the same sequences scored by two checkpoints, threaded between slabs |
| Five ways to write the same protein | where each rewrite family lands relative to the boundary |
| The walk across the boundary | one rewrite applied in eighths, as a path |

Recurring axes:

- **class axis** is position along the line between the two class centroids, so 0 is the viral
  centroid and 1 the host centroid.
- **P(viral)** is the classifier's own output, with the decision boundary at 0.5.
- **depth** differs per scene: sometimes a measured direction, sometimes a categorical slot.
  Each scene's footnotes say which, and one of them says outright that its depth is a family
  label rather than a measured direction.

Points are precomputed, not live inference. Frames are never merged: each scene keeps the axes
it was computed in, so coordinates are comparable within a scene and not across scenes. The
in-app footnotes carry the rest of the per-scene caveats.

## Bundle format

The viewer reads one JSON object from an inline `<script type="application/json" id="bundle">`
tag. `latent_points.json` in this repo is that same object as a standalone file. To view your
own data, replace the inline JSON with a bundle of the same shape:

```json
{
  "scenes": [{
    "id": "frame",
    "title": "The decision frame",
    "lede": "One or two sentences under the title.",
    "axes": {
      "x": {"label": "class axis", "range": [-0.75, 1.3],
             "refs": [{"at": 0, "text": "viral centroid"}]},
      "y": {"label": "leading synonymous direction", "range": null},
      "z": {"label": "P(viral)", "range": [0, 1],
             "refs": [{"at": 0.5, "text": "decision boundary"}]}
    },
    "boundary": 0.5,
    "pairs": true,
    "footnotes": ["Shown under the fold, one per line."],
    "series": [{
      "key": "viral",
      "label": "Viral, held out",
      "color": "#ac017a",
      "symbol": "circle",
      "size": 4,
      "opacity": 0.45,
      "role": "reference",
      "points": [{"x": 0.718, "y": 2.87, "z": 0.969, "pi": null}]
    }]
  }]
}
```

`role` drives styling and legend grouping. `pi` is an optional parent index: when a scene sets
`pairs: true`, points sharing a parent are linked on hover. A scene may also carry a `links`
array of precomputed line segments or paths, which is what the second and fourth scenes use to
draw their threads.

Rendering is [Plotly](https://plotly.com/javascript/) 3.0.1, loaded from a CDN, so the page
needs network access on first paint. Everything else is inline.

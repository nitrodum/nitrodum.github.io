Exactly — the Pretext technique from the video: sample pixel brightness from the rendered name, fill a word grid, bright pixels = visible words, dark pixels = faint. Then layer cursor physics on top. Let me build it.Created a file, read a fileCreated a file, read a fileHere's exactly how it works, mirroring the video technique:
The Pretext-inspired approach:

"DEREK KWOK" is rendered in bold Impact onto an invisible offscreen canvas
Every cell in a word grid samples the brightness at that pixel position
Bright pixels (the letter strokes) → full opacity words; dark pixels (background) → nearly invisible at 4.8% opacity
Two batched draw passes per frame (background cells, then name cells) — only 2 globalAlpha state changes total, so it runs smooth at 60fps

The physics:

Each word cell has position + velocity + spring force pulling it back to its origin
Cursor creates a quadratic falloff repulsion field — strongest at centre, tapering to nothing at the edge (120px radius). This gives the forcefield feeling rather than a hard wall
DAMPING 0.78 means slight oscillation when words return, which is what gives it the fluid/water feel
Moving slowly through the name creates a parting-water effect; moving fast creates a chaotic scatter

To tune the feel, just change the constants at the top of the JS:

More dramatic scatter: raise FORCE (try 12–14)
Bigger forcefield: raise RADIUS (try 160)
More bouncy/watery: lower DAMPING (try 0.68)
Snappier return: raise SPRING (try 0.08)

To deploy, drop test.html alongside index.html — Netlify will serve it at /test.
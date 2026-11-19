# elementary-school-teacher

**Explains your code so your grandma gets it. And so do you.**

The most patient senior dev in the room, as an agent skill.

## Real example

Question: *"how does glass work in this renderer?"*, asked about a ray tracer written in Rust, without and with the skill. Both answers are real and unedited, each from a fresh session.

**With the skill**

> The renderer draws a picture by sending out lines of sight, one per pixel, and asking "what does this line hit?" Glass is special: when a line of sight hits it, the line splits in two.
> Here's what happens when a line of sight hits glass:
>
> 1. It splits. One copy bounces off the surface, like in a mirror. The other copy goes into the glass.
> 2. The copy that goes in bends. Each material has a "bending strength" (a number for how much it bends light). Glass bends light more than air does. So the line changes direction as it goes in, and again as it comes out.
> 3. The renderer keeps track of what the line is inside. It might be air, glass, or glass inside water. That tells it which two bending strengths to compare at each surface.
> 4. The angle decides the mix. Look at glass straight on and you mostly see through it. Look at it from a low, grazing angle and it acts more like a mirror. The renderer blends the bounced color and the see-through color using that angle.
> 5. Sometimes light gets trapped. Inside glass, if the line hits the surface at too shallow an angle, it can't get out and only bounces. Real glass does this too.
>
> Every split creates new lines, and those can split again. So the renderer stops after a set number of bounces to avoid going forever.
> There's more detail if you want it, like how the lines start just inside or just outside the surface so they don't hit the same spot again by mistake. Want the same walkthrough at junior level, with the real code names?

<details>
<summary>Without the skill (click to expand)</summary>

Glass is a Whitted-style recursive ray tracer that follows The Ray Tracer Challenge. It isn't a special shader. It's a regular Phong material with three settings, plus some recursion in World::shade_hit.

1. The material
materials.rs:35-41:

```rust
pub fn glass() -> Self {
    let mut material = Material::phong();
    material.transparency = 1.;
    material.reflective = 1.;
    material.reflactive_index = 1.5;   // (sic: refractive index)
    material
}
```

The scenes then make it nearly invisible as a surface. cena_video.rs:53-58 sets low ambient and diffuse, high specular and shininess, and a slight blue tint. So almost all of what you see comes from the reflected and refracted rays, not from Phong shading.

2. Working out n1 and n2 for each hit
intersection.rs:79-107 (prepare_computations) goes through every intersection along the ray, sorted by t, and keeps a containers stack of the objects the ray is currently inside. A hit pushes the object onto the stack, and a second hit on the same object pops it off.

- n1 is the index of the top object on the stack just before the hit (1.0 if the stack is empty, meaning air).
- n2 is the index of the top object just after the hit.

This makes nested or overlapping transparent objects work, such as an air bubble inside glass.

Computations::from also builds two offset points:

- over_point (point + normal·ε) starts reflection and shadow rays.
- under_point (point − normal·ε) starts the refraction ray just inside the surface, so it doesn't hit the same surface again.

If the normal points away from the eye, the hit is from inside the object. In that case inside = true and the normal is flipped.

3. Refraction (Snell's law)
world.rs:74-90:

- n_ratio = n1/n2, cos_i = eye·normal, sin²t = n_ratio²·(1 − cos_i²)
- If sin²t > 1, that's total internal reflection, and the refraction ray returns black. The light continues through the reflection ray instead.
- Otherwise the new direction is normal·(n_ratio·cos_i − cos_t) − eye·n_ratio. The ray starts at under_point, and its color is multiplied by transparency.

4. Fresnel blend (Schlick)
In world.rs:56-60, if a material is both reflective and transparent, the reflected and refracted colors are blended instead of added:

```rust
surface + reflected * R + refracted * (1 - R)
```

R comes from Computations::schlick (intersection.rs:48-61):

- r0 = ((n1−n2)/(n1+n2))²
- R = r0 + (1−r0)(1−cos)⁵

When going from a denser to a less dense medium (n1 > n2), it uses cos_t instead and returns 1.0 under total internal reflection. This is why glass edges look like mirrors while the centre is see-through.

5. Recursion limit
Each reflection or refraction ray calls color_at(ray, remaining - 1). When remaining reaches 0, it returns black. A ray that misses everything also returns black, because there's no environment map.

Limitations
- Glass casts solid shadows. is_shadowed (world.rs:92-106) treats any hit closer than the light as a full shadow, whatever the material's transparency. There's no caustics or light transmission.
- No absorption. Thicker glass isn't darker or more tinted (no Beer–Lambert). The color tint only affects the Phong surface term.
- No dispersion or rough glass. One index of refraction for all wavelengths, and perfectly sharp refraction.
- Possible issue with grouped meshes. n1/n2 are tracked by comparing Object equality. For a triangle mesh like the teapot, each triangle is probably its own Object. If so, the containers stack sees the entry and exit as different objects, and n1/n2 inside the mesh could be wrong. I haven't checked how Object equality behaves for BVH triangles, so this is a guess, not something I confirmed.

</details>

What changed:

- Fresnel, refractive index and total internal reflection are all still there, explained without jargon: "the angle decides the mix", "bending strength", "light gets trapped".
- One idea per step, instead of file paths, line numbers and formulas in every paragraph.
- It's still correct: every step matches what the code actually does.

## The problem

Ask an AI agent to explain a project, a workflow or a PR, and you get dense jargon, acronyms and implementation detail. Only an expert can follow it.

This skill makes the agent explain like a great elementary school teacher: short, concrete, never condescending, and still correct.

## What it does

- Big picture first: what it is and why it exists, before how it works.
- Everyday words. Unavoidable terms get defined once, in plain words.
- One analogy from daily life, when it helps. Never a pile of them.
- Flows told as a short numbered story.
- Simplifies, but never lies. If something important is left out, it says so.
- Answers what you asked, then offers to go one level deeper.

## Install

**Claude Code: copy the skill folder (works everywhere: Windows, desktop app, IDE, terminal)**

Windows (PowerShell):

```powershell
git clone https://github.com/ViniciusSJV/elementary-school-teacher
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills" | Out-Null
Copy-Item -Recurse elementary-school-teacher\skills\elementary-school-teacher "$env:USERPROFILE\.claude\skills\"
```

macOS / Linux:

```bash
git clone https://github.com/ViniciusSJV/elementary-school-teacher
mkdir -p ~/.claude/skills
cp -r elementary-school-teacher/skills/elementary-school-teacher ~/.claude/skills/
```

Start a new session and the skill is available.

**Claude Code in the terminal: plugin (optional)**

If your Claude Code has the `/plugin` command:

```
/plugin marketplace add ViniciusSJV/elementary-school-teacher
/plugin install elementary-school-teacher@elementary-school-teacher
```

**Codex**

Copy [`AGENTS.md`](AGENTS.md) into your project root. To use it everywhere, add its content to `~/.codex/AGENTS.md`.

**Cursor**

Copy [`.cursor/rules/elementary-school-teacher.mdc`](.cursor/rules/elementary-school-teacher.mdc) into your project's `.cursor/rules/` folder.

**Windsurf**

Copy [`.windsurf/rules/elementary-school-teacher.md`](.windsurf/rules/elementary-school-teacher.md) into your project's `.windsurf/rules/` folder.

**Any other agent**

Copy [`AGENTS.md`](AGENTS.md) into your project, or paste it into the agent's rules or system prompt.

## Usage

It kicks in when you ask for an explanation:

- "explain this", "what does this do"
- "analyze this project / workflow / code"
- "I don't understand", "ELI5", "in plain words"
- "summarize this repo"
- onboarding someone non-technical

In Claude Code you can also call it directly: `/elementary-school-teacher`.

### Depth levels

| Level | Reader |
|-------|--------|
| `kid` | A 10-year-old could follow it. |
| `newcomer` | A smart person who isn't a programmer. **Default.** |
| `junior` | A developer new to this codebase: real terms, each one defined. |

Switch with `/teacher junior` (in Claude Code: `/elementary-school-teacher junior`), or just say "explain it at kid level".

### Turn it off

Say "stop teacher", "normal mode", or `/teacher off`.

## Files

- [`skills/elementary-school-teacher/SKILL.md`](skills/elementary-school-teacher/SKILL.md): the prompt. Source of truth.
- [`AGENTS.md`](AGENTS.md): compact version for any agent that reads a rules file.
- `.cursor/` and `.windsurf/`: the same rules as `AGENTS.md`, in each editor's format.

## Credits

Inspired by [Ponytail](https://github.com/DietrichGebert/ponytail), "the laziest senior dev in the room": one strong, opinionated prompt and no bloat.

## License

[MIT](LICENSE)

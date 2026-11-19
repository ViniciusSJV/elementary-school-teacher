# elementary-school-teacher

**Explains your code so your grandma gets it. And so do you.**

The most patient senior dev in the room, as an agent skill.

## Real example

Question: *"how does glass work in this renderer?"*, asked about a ray tracer written in Rust, on the same model, in fresh sessions with and without the skill. Both answers are real and unedited.

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

Glass works the way it does in The Ray Tracer Challenge (chapter 11). It isn't its own material type. It's an ordinary Phong material with three fields set, and the ray tracer recursively traces a reflected ray and a refracted ray, then blends them using Fresnel.
1. The material: materials.rs:35-41

```rust
pub fn glass() -> Self {
    let mut material = Material::phong();
    material.transparency = 1.;
    material.reflective = 1.;
    material.reflactive_index = 1.5;   // (sic: "reflactive")
    material
}
```

Any material with `transparency > 0` behaves like glass. Scenes adjust it further. For example, cena_video.rs:53-58 adds a light blue tint, lowers `diffuse` and `ambient` so the surface shading doesn't wash it out, and raises `shininess` for tight highlights.
2. Finding n1 and n2: intersection.rs:76-105
To refract a ray you need the refractive index on each side of the surface. `prepare_computations` walks the sorted list of intersections and keeps a stack (`containers`) of the objects the ray is currently inside:

* n1 is the index of the innermost object just before the hit, or 1.0 (air) if the ray isn't inside anything.
* Then the hit object is pushed onto the stack (entering) or popped off it (leaving).
* n2 is the index of the innermost object just after the hit.

Because of this, nested objects work, like an air bubble inside a glass sphere or glass inside water.
It also computes `under_point = point - normal * EPSILON` (intersection.rs:38). The refracted ray starts just below the surface so it doesn't hit the same surface again (acne). Reflected rays use `over_point` instead.
3. Refracted ray (Snell's law): world.rs:74-90

```rust
let n_ratio = n1 / n2;
let cos_i   = eye_v · normal_v;
let sin2_t  = n_ratio² * (1 - cos_i²);
if sin2_t > 1 { return black }          // total internal reflection
let cos_t   = sqrt(1 - sin2_t);
let direction = normal_v * (n_ratio*cos_i - cos_t) - eye_v * n_ratio;
color_at(Ray::new(under_point, direction), remaining - 1) * transparency
```

Under total internal reflection, refraction adds nothing and only the reflected ray carries light. `remaining` is the recursion budget. When it reaches 0, the function returns black.
4. Combining the parts with Fresnel: world.rs:45-63
`shade_hit` adds up the Phong surface color, the reflected color and the refracted color. If the material is both reflective and transparent (as glass is), it weights the two rays with Schlick's approximation:

```rust
surface + reflected * reflectance + refracted * (1 - reflectance)
```

5. Schlick: intersection.rs:48-61
`r0 = ((n1-n2)/(n1+n2))²`, then `r0 + (1-r0)(1-cos)^5`. When the ray goes from a denser medium to a lighter one (n1 > n2), it uses `cos_t` instead of `cos_i`, and it returns 1.0 under total internal reflection. The result is that glass looks transparent when you look at it straight on and becomes mirror-like at grazing angles (the edges of the teapot).
Limitations

* Shadows: world.rs:92-107 counts any hit as blocking the light, so glass casts a fully black shadow just like an opaque object. A common fix is to skip objects with `transparency > 0` in `is_shadowed`, or scale the shadow by transparency.
* No absorption or dispersion: the tint comes only from the surface `color` in the Phong term. Thicker glass doesn't get darker or more colored, and there are no prism or rainbow effects.
* No caustics: this is a Whitted-style ray tracer, so light focused through glass onto the floor isn't simulated.
* Rays that miss everything return black (`color_at`), so whatever is behind the glass needs a backdrop in the scene, or the glass looks dark.

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

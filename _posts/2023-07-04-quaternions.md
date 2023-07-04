---
title: Quaternions
date: 2023-07-04 12:00:00 -0700
categories: [Blogging]
tags: [math,C#,unity]     # TAG names should always be lowercase
image: https://hugelolcdn.com/hugewoah.com/i/7950.gif
math: true
img_path: /imgs/
---

> <center><i>Here as he walked by on the 16th of October 1843 Sir William Rowan Hamilton</center>
> <center>in a flash of genius discovered the fundamental formula for quaternion multiplication:</center>
> <center><code>i<sup>2</sup> = j<sup>2</sup> = k<sup>2</sup> = ijk = -1</code></i></center>
> <center>—Plaque on Broom Bridge, Dublin</center>

## Intro

So you want to learn about quaternions? Well, you've come to the right place. I'll try my best to simplify it for you. This is written assuming that you know the following:
- Vectors and/or Vector math fundamentals,
- the difference between "global" and "local", or the term "relative to" **(this is important)** 

I've split this article into four main parts:
1. **[Intuition]**. How to think about them
2. **[Properties of Quaternions]**
3. **[Unity Methods]** and alternative rotational options
4. Deeper dive into the **[Math]** behind them

But first, let's go over some **Terminology** before we get started.

You probably know already that a quaternion represents a "rotation". Great! However, the term "rotation" is a bit overloaded, so to clarify the terminology I'll be splitting it into two distinct terms (Note: these are not "official" or universally standardized terms):
- <u>Orientation</u>: the **state** or **pose** of an object. How the object is ~~positioned~~/oriented in space.
- <u>Rotation</u>: the **action** or **verb** of rotating/turning/spinning an object. Specifically denotes the transformation or struct that will be *applied* to another **rotation *or* orientation**.

<details><summary><b>Example Terminology Usage</b></summary>
Consider an object initially facing East. "Facing East" represents its current orientation. To make the object face East again after a change in orientation, we need to apply a rotation. On the other hand, if we simply say "turn 90° about the Y axis", we directly specify the rotation without explicitly mentioning the resulting orientation. English is hard; hopefully this helps.
</details>

Also worth mentioning...
- I will be using Unity's coordinate system (X = right, Y = up, Z = forward), which is a Left-Handed coordinate system (i.e. satisfies Left-Hand Rule). If you don't know what that means, look it up. In short: positive angles are CCW rotations when looking in the same direction of the axis you're rotating about.

## Intuition

### Frame of Reference

Are you looking to perform a *relative* rotation or a *global* one? Just like how a vector can represent either a global or local position, a quaternion struct can be global or local - it matters how you set them up and use them. Without any context, you can consider all quaternions to be global because you have nothing else to reference the rotation on. However, if you set up a quaternion using a relative value and apply it to the same object, that makes it a relative rotation. For instance, using `transform.forward` would make the rotation relative to the object's `Transform`, while `Vector3.forward` represents a global rotation.

It's generally not a good idea to try to share relative rotations with another object, especially if they started with a different orientation.

Don't confuse `transform.rotation` and `transform.localRotation`. Know the difference.<br>
Are you calculating a global orientation relative to the global XYZ coordinates (e.g. wanting to make your character face a target), then use the global-facing `transform.rotation`. If you only care about how it rotates with respect to its parent (e.g. Animations, door swinging on a hinge), then use `transform.localRotation`.

Bonus Tip: In Unity, toggle between "Local" and "Global" to see which axis is which on the Gizmo. Extra useful if you imported a model and the axes are set up differently.

### Axis and Angles

Look around yourself at all the things that do any kind of rotating. And I want you to think in terms of **Axis** and **Angles**. That's a singular axis: a vector in any direction of your choice. This axis will be what gets rotated *around* by a certain amount of angular units. I'll use degrees, but radians are valid too.

Leaning back in your chair? Nah. You're rotating the chair negative degrees about its `right` axis (or positive degrees about the `left` axis).

Looking down and to your left? Nah. Your neck is oriented some positive degrees around `Vector(1, -1, 0)`, assuming Z is your forward. Might be tough to wrap your head around.

Grab an apple :apple:. Grab a pen :pen:. Make an Apple Pen :apple: :pen:. That pen is the axis, at whatever arbitrary vector you stuck it in at. Spin it to see how it rotates.

Why think in terms of axis and angles? Because that's more in line with how quaternions actually rotate. If you have a start and end orientation and want to interpolate between them, the shortest path is achieved with this kind of visualization. It's like drawing two dots on a basketball and connecting it with the shortest line. Look at the path the dot needs to travel, and rotate it with an axis that's perpendicular to that.

### Why Euler Angles Suck

*Disclaimer: These reasons don't really apply to standard 2D games, because you usually only rotate thing about a single Z axis. So it's typically fine to use Euler Angles there.*

<details><summary>What are Euler Angles?</summary>
It's a combination of 3 numbers representing the amount of angles rotated about each axes (X, Y, and Z), with a total range of 360° each. This is what you're used to using and seeing and what's displayed in Unity's Inspector for a Transform's "Rotation".
</details>

1. You can't always trust the numbers you see in the Inspector

Those numbers next to "Rotation" aren't even its `transform.rotation`, but rather the Transform's **local orientation** (i.e. relative to its parent). The Inspector lies to you, so don't be fooled.

Despite displaying as Euler Angles, they're actually a Quaternion behind the scenes. If you rotate around multiple axes, like for some 3D games, don't use its Euler Angles as a reference to base your logic off of. This is because the internal conversion that Unity does between Quaternions and Euler Angles can have some of the angles "jumping around", giving you **unpredictable** and **unreliable** behavior.

2. Euler Angles are subject to Gimbal Lock

"What even is that?" you might be thinking. Let me Google that for you: "Gimbal Lock happens when two of the rotational axes align, causing a loss of one degree of freedom." Okay, that wasn't very helpful.

But it's fairly easy to demonstrate with an actual example, so let's do a "show, don't tell" approach here. Put the following script on a GameObject:

```csharp
public class GimbalLockExample : MonoBehaviour
{
    [SerializeField] private float _speed = 50f;
    private float _pitch, _yaw, _roll;

    private void Update()
    {
        _pitch -= Input.GetAxis("Vertical") * _speed * Time.deltaTime; // W/S
        _yaw += Input.GetAxis("Horizontal") * _speed * Time.deltaTime; // A/D
        var leftRoll = Input.GetKey(KeyCode.Q) ? 1f : 0f;
        var rightRoll = Input.GetKey(KeyCode.E) ? -1f : 0f;
        _roll += (rightRoll + leftRoll) * _speed * Time.deltaTime; // Q/E

        transform.rotation = Quaternion.Euler(_pitch, _yaw, _roll);
    }
}
```
Pitch upwards by pressing W about 90° so that the forward vector aligns with the global Y-axis. Then any adjustment to Yaw (A/D) or Roll (Q/E) will be indistinguishable from one another; they both behave like Roll. That's a loss of a degree of freedom. That's gimbal lock.

3. Interpolation sucks

At the end of the [Axis and Angles] section, I pointed out that quaternions will interpolate along the shortest path. If you interpolate with Euler Angles, well... it's not a straight path. It will be more like a soft "S". Or it will just glitch out the rotation entirely, because Lerping one axis from 359° to 0° *doesn't* move just 1° like you'd hope it would.

4. Dealing with the wrap-around point sucks

Having to add extra code to handle the transition between 0° and 360° (or -180° and 180°) to complete the circle is tedious. This is a situational game-dependent opinion, and sometimes unavoidable, but it's still a pain. Clamping how far up and down you can look in an FPS is fine. But managing Eulers for handling Yaw in a Top-Down/3rd Person kind of game seems wrong to me.

## Properties of Quaternions

When you see or hear "multiplying", it's easier to think of "applying" instead. A quaternion times a vector means you're *applying* the rotation *to* the vector. If you ever heard someone say *"multiplying quaternions together is like adding them"*, **unlearn that**. We don't "add" quaternions in the traditional sense. Instead, one rotation gets *applied* to the other. It's better in the long-run to think about them properly rather than lazily.

### Unit Quaternions (Normalized)

Unit quaternions have a ||magnitude|| of 1. Within Unity, quaternions get normalized by default. There does exist a `.Normalize()` method in case you're manually entering quaternion values and it, for some reason, doesn't auto-normalize. But usually you shouldn't have to worry about that. Non-Unit Quaternions are more complicated things that I'm just not going to get into.

### Identity ("1")

Identity just means "no change". It's like multiplying regular numbers by `1`, or adding `0`. When you set an object's orientation to `Quaternion.identity`, it goes to the default orientation, which represents an absence of any rotations applied (i.e. 0° on all Euler Angles).

### Invertability (Ctrl + Z)

The inverse of a quaternion `Q` is denoted as <code>Q<sup>-1</sup></code>. Any rotation that is applied can be undone. A quaternion will cancel out with its inverse. Thinking in terms of *Axis and Angles*, you can view it as "the opposite rotational angle with same axis direction" or "the same rotational angle with an opposite axis direction". Same end result. Mathematically speaking:

<code>Q * Q<sup>-1</sup></code> = <code>Q<sup>-1</sup> * Q</code> = 1 (Identity)

### Associativity (Parentheses order doesn't matter)

Given quaternions <code>Q<sub>1</sub></code>, <code>Q<sub>2</sub></code>, and <code>Q<sub>3</sub></code>, associativity states that:

<code>(Q<sub>1</sub> * Q<sub>2</sub>) * Q<sub>3</sub></code> is the same as <code>Q<sub>1</sub> * (Q<sub>2</sub> * Q<sub>3</sub>)</code>

Same applies when multiplying with a vector `V`:

<code>(Q<sub>1</sub> * Q<sub>2</sub>) * V</code> is the same as <code>Q<sub>1</sub> * (Q<sub>2</sub> * V)</code>

### Non-Commutativity (Multiplcation order matters)

Given quaternions <code>Q<sub>1</sub></code> and <code>Q<sub>2</sub></code>, non-commutativity states that:

<code>Q<sub>1</sub> * Q<sub>2</sub></code> is **not** always the same as <code>Q<sub>2</sub> * Q<sub>1</sub></code>

There are *some cases* where they will just happen to be equal. Like if at least one of them is the Identity quaternion, if one is the Inverse of the other, or some other exceptional case involving symmetry.

When multiplying quaternions together, the best way to think about it is from **right-to-left** (I know, it seems backwards). Given <code>Q<sub>1</sub> * Q<sub>2</sub> * V</code>: start with the Vector direction, rotate it *first* by <code>Q<sub>2</sub></code>, *then* rotate that resulting vector by <code>Q<sub>1</sub></code>. Order of operations is important to visualize the rotations.

## Unity Methods

### Quaternion Creation

- <a href="https://docs.unity3d.com/ScriptReference/Quaternion.Euler.html">Euler</a>: Rotation that rotates `z°` around the `z` axis, `x°` around the `x` axis, and `y°` around the `y` axis. **In that order**
    - Can access these values with `transform.eulerAngles` - not to be confused with `transform.rotation`
    - If you're typically setting two of the three values to `0°`, you can accomplish the same thing with AngleAxis instead
- <a href="https://docs.unity3d.com/ScriptReference/Quaternion.AngleAxis.html">AngleAxis</a>: Are you thinking in terms of axis and angles yet?
    - You can create most of the same quaternions you'd probably make with Euler with this instead
    - This method is ***FASTER***, too! With a basic test of 100,000 calls every frame, AngleAxis outperformed Euler with 30-50% more FPS. Ditch using Euler; it's a more expensive conversion
- <a href="https://docs.unity3d.com/ScriptReference/Quaternion.LookRotation.html">LookRotation</a>: Get an orientation by providing a `Vector3 forward` and a `Vector3 upwards` as a "hint" to determine Roll. Default hint is `Vector3.up`
    - Usually you'll get a forward direction by doing `target.position - transform.position`
    - Don't have these two vectors align. If you're looking straight up or down, provide a different upwards hint
    - <details><summary>Example Analogy</summary>Imagine you are an astronaut in space and were told to fixate your eyes in a certain direction. Let's assume you could accomplish that part. Great. But there wasn't a strict instruction on how to orient the rest of your body, so you're spinning around 360° while your eyes stay focused. But with an "upwards" reference direction, that's enough to narrow down your possible orientations to just 1 and stop your spinning. Your personal "up" might not be exactly parallel with the "upwards" hint, but it's enough info to tether you to something for some stability</details>
- <a href="https://docs.unity3d.com/ScriptReference/Quaternion.FromToRotation.html">FromToRotation</a>: This gets the difference in orientation between two direction vectors; the *rotation* that you would need to apply to `fromDirection` which will result in the `toDirection`.
    - In other words, this solves for <code>Q<sub>FromTo</sub></code> in the equation <code>Q<sub>FromTo</sub> * V<sub>From</sub> = V<sub>To</sub></code>
    - This method was vital for my Roll-a-Tetrahedron solution
- <a href="https://docs.unity3d.com/ScriptReference/Quaternion.Inverse.html">Inverse</a>: Gets the ~~evil twin~~ opposite rotation of a quaternion.
    - Example: if you wanted to get <code>Q<sub>ToFrom</sub></code>, you *could* call FromToRotation again swapping parameters or you could just do <code>Q<sub>ToFrom</sub> = Quaternion.Inverse(Q<sub>FromTo</sub>);</code>
    - Very simple method. Given some `Q`, <code>Q<sup>-1</sup></code> is just <code>new Quaternion(Q.x, Q.y, Q.z, -Q.w);</code>

### Quaternion Interpolation

When interpolating between a start and end orientation, Quaternions take the shortest path (this means that angles more than 180° apart aren't a thing). Try to avoid having your start and end 180° apart (pointing in opposite directions), otherwise the path between them will not be predictable.

#### Time- or Percent-Based Methods
- <a href="https://docs.unity3d.com/ScriptReference/Quaternion.Lerp.html">Lerp</a>: Linear interpolation between [0-1]
- <a href="https://docs.unity3d.com/ScriptReference/Quaternion.Slerp.html">Slerp</a>: Spherical interpolation between [0-1]
- <a href="https://docs.unity3d.com/ScriptReference/Quaternion.LerpUnclamped.html">LerpUnclamped</a> + <a href="https://docs.unity3d.com/ScriptReference/Quaternion.SlerpUnclamped.html">SlerpUnclamped</a>: can also extrapolate beyond 0 and 1

<details><summary>What's the difference between Lerp and Slerp? Aren't they both spherical since it involves things that rotate?</summary>
Yeah, not much difference tbh. Unlike the differences between Vector3's Lerp and Slerp, these all follow the same path but just have an ever-so-slightly different easing/timing along the path. The docs say that "[Lerp] is faster than Slerp but looks worse if the rotations are far apart." But it's hard to see a difference</details>

<details><summary>Do the Unclamped versions behave the same?</summary>
<b>No</b>. Because of how the math works with extrapolating and quaternions getting auto-normalized, LerpUnclamped "fizzles out" at certain values outside the 0-1 range, like diminishing returns. Unsatisfying.<br>In my tests, if the angle between the start and end orientations are less than ~36° apart, then SlerpUnclamped <i>also</i> behaves like LerpUnclamped. However, at larger starting angles, SlerpUnclamped shines and will properly extrapolate.<br>For example, if you wanted to extrapolate the Minute Hand on a Clock, your start and end orientations should be at least 6 minutes apart, but less than 30 minutes so that <i>forward</i> in time doesn't go the wrong way
</details>

#### Speed-Based Method
- <a href="https://docs.unity3d.com/ScriptReference/Quaternion.RotateTowards.html">RotateTowards</a>: Get an orientation that is the `from` quaternion rotated by a `float maxDegreesDelta` angle towards the `to` quaternion without overshooting
    - If you use a `float rotationSpeed` multiplied by `Time.deltaTime`, it will rotate at that **rate** in degrees per second, no matter the difference between `from` and `to`

### Transform Options for Rotating

Quaternions by theirself are just a math construct - they have no idea what a Transform is. With the Transform class, you have access to more contextual utility, like methods that know the difference between global and local.

- <a href="https://docs.unity3d.com/ScriptReference/Transform.Rotate.html">Transform.Rotate</a>: This method has many overlaod options
    - Primarily relies on Euler Angles or an Axis and Angle combo
    - Can specify whether it's a global (`Space.World`) or local (`Space.Self`) rotation (default)
- <a href="https://docs.unity3d.com/ScriptReference/Transform.RotateAround.html">Transform.RotateAround</a>: Like a satellite orbiting around a planet or like a hinge joint, this method **also affects the position**
    - Uses an Axis and Angle and a world position `Vector3 point` as a pivot or anchor that the axis passes through
- <a href="https://docs.unity3d.com/ScriptReference/Transform.LookAt.html">Transform.LookAt</a>: Immediately snap a transform's forward to a `Transform target` or `Vector3 worldPosition`
    - Similar to `Quaternion.LookRotation`, this also can take a `Vector3 worldUp` hint
    - For 2D games, since this defaults to using the transform's forward, you'll want to adjust the `worldUp` to have it orient towards a certain direction, leaving forward as `Vector3.forward`
- Setting a `Vector3 direction` directly to <code>transform.<a href="https://docs.unity3d.com/ScriptReference/Transform-forward.html">forward</a>/<a href="https://docs.unity3d.com/ScriptReference/Transform-up.html">up</a>/<a href="https://docs.unity3d.com/ScriptReference/Transform-right.html">right</a></code>
    - This approach doesn't give control over where the other axes choose to align, so use cautiously or only in very simple cases
- Setting a `Quaternion rotation` directly to <a href="https://docs.unity3d.com/ScriptReference/Transform-rotation.html">`transform.rotation`</a> or <a href="https://docs.unity3d.com/ScriptReference/Transform-localRotation.html">`transform.localRotation`</a>

### Quaternion Methods You'll Never or Rarely Use

- <a href="https://docs.unity3d.com/ScriptReference/Quaternion.Normalize.html">Normalize</a>: Set the magnitude of a quaternion to 1, keeping its orientation. Unity does this by default in most cases
- <a href="https://docs.unity3d.com/ScriptReference/Quaternion.Dot.html">Dot</a>: Returns the Dot Product between two quaternions, a value between -1 and +1 as a measure of "alignment". Easier to use Vectors
- <a href="https://docs.unity3d.com/ScriptReference/Quaternion.Angle.html">Angle</a>: Returns the angle in degrees between two quaternions. Easier to use Vectors
- <a href="https://docs.unity3d.com/ScriptReference/Quaternion.ToAngleAxis.html">ToAngleAxis</a>: Gets an Angle and Axis from a quaternion. Usually you're using that kind of info to create quaternions, not the other way around
- The Quaternion <a href="https://docs.unity3d.com/ScriptReference/Quaternion-ctor.html">Constructor</a>, taking in x,y,z, and w: If you're using this, it's either for the purpose of deserialization, optimizations (e.g. swizzling via extension methods), you understand quaternions better than most, or you copied code you found somewhere

## Math

Quaternions are a four-dimensional struct. As with all multi-dimensional things, all the axes are perpendicular to each other. If that's hard to imagine, three of its four axes are imaginary.
- Imaginary numbers
- Complex plane


To create a Quaternion using the default constructor is actually really easy, but is almost never needed. They require four floats: x,y,z, and w, which range between negative one and one. It will also need to be normalized, so that its magnitude is always 1. Just like AngleAxis from a few seconds ago, the x,y,z will represent the axis and w will represent the angle. But we need to do some remapping to fit within the -1 to 1 range. 
Here are the conversion formulas:
To Quaternion (Given: angle, axis)
From Quaternion (Given: x,y,z,w)
w = cos(angle / 2)
Angle = 2 * acos(w)
var remapped = axis.normalized * sin(angle / 2) 
Axis = new Vector3(x,y,z); // normalize optional
var Q = new Quaternion(x, y, z, w); 
// x,y,z from remapped vector
// or just used Q.ToAngleAxis(out float angle, out Vector3 axis);



i2 = j2 = k2 = -1
ij = k
ji = -k
jk = i
kj = -i
ki = j
ik = -j



 - Notation
 - Rules & properties (non-commutative)
 - QxQ
 - QxVxQ-1

Misc Bonus
 - 720 degrees
 - Freya swizzle shortcuts

Still confused? Resources
 - links to 3b1b
 - that one good other video


The inverse can be achieved in two steps. Grab the angle by multiplying 2 and the result of arcosine w, the axis can be obtained simply by extracting the xyz



QxV Multiplication
You can multiply a quaternion and a Vector3 together, which will result in a Vector3 that has been rotated by the quaternion, which is to say: an angle around an axis. It must be in that order (QxV), not vector x quaternion. 
QxQ Multiplication
A thing to note is when multiplying quaternions together, it is non-commutative, meaning that AxB does not equal BxA, so order of operations matters.
To envision the result of multiplying a quaternion by another, think of it from right to left: We start right the right-most quaternion as an orientation, then go to the left one at a time and apply that rotation to it [is it worth emphasizing again here that they use the “global” axes?]
Pro Tip: If you want to adjust a transform’s rotation, 
Do this: transform.rotation = myQuaternion * transform.rotation;  
Don’t do this: transform.rotation *= myQuaternion; 


Multiplication
Setup / Intro / Foundation:
Multiplying by a pure imaginary number (i) has the effect of rotating 90° (kind of like a cross-product). [Visuals: 2D graph of the complex plane. 1 -> i -> -1 -> -i ]
Let’s expand this 2D graph by two more dimensions, adding a “j” and “k” which both have the same property of being imaginary [Can’t easily visualize 4D stuff, but it would kind of look like the right image minus all the transition lines/arrows]
Since all 4 axes are perpendicular, there are some important rules (properties?) to follow when multiplying 1 axis by another. [Visuals: Display rules (from table below, but color-coded & pretty), which can then be combined into a multiplication table]


Side Note: I’ve been putting 1 at the end instead of the beginning just to match Unity’s order of xyzw (examples further down below). You can choose either order, whether you want to match unity or math convention.
(Q*Q) Example: (TODO: come up with a good rotation example; two nice quaternions)
Let’s take two quaternions and multiply them. [Describe the rotations, show their xyzw values, then put them along a multiplication table. Color-code results]
As far as the math goes, it’s just basic multiplication and addition. [Animate combining the 16 values together, resulting in 1 quaternion xyzw]
[Prove that it’s been rotated correctly by converting it to the friendlier angle & axis]
(Q*V) Example: (Similar TODO: can use a similar quaternion from before + new vector. Suggestion: go with a bigger vector, one that’s not normalized)
Two important differences you must know in order to involve Vectors:
1. They live in a different space (real world vs imaginary land). We will “pretend” that the vector is a quaternion with a “w” of 0, and that all 3 axes are imaginary.
2. The importance of inverses. Q*Q-1 = 1. (If we just carried out the multiplication, we’d end up with a quaternion just like in the prev Q*Q example. But we want to guarantee we get w=0 at the end…) So the Vector gets sandwiched like so: Q*V*Q-1.
As always with quaternion multiplication, the positioning matters (is not commutative) but where you put the parentheses doesn’t matter (is associative):  (QV)Q-1 = Q(VQ-1) [Carry out two sets of multiplication. Final sum should have w=0. Drop w and the imaginary letters to bring it back to reality and you’ve got your result]



https://docs.unity3d.com/Manual/QuaternionAndEulerRotationsInUnity.html

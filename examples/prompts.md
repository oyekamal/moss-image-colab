# Prompts and settings

All pictures: model `OpenMOSS-Team/MOSS-Image-SFT`, free Colab T4, seed 42, guidance scale 3.0, 30 steps, float16, image model in int8 and text encoder in 4-bit (the recipe in the notebook).

Negative prompt for the Marko pictures:

```
cow, bull, ox, yak, calf, snout, muzzle nostrils, tail, plain curved horns, realistic, photo, 3d, shading, gradient, watermark, extra limbs
```

The notebook itself uses a shorter one: `realistic, photo, 3d, shading, gradient, watermark, extra limbs`.

## Marko description (shared by the Marko pictures)

```
a baby markhor mountain goat mascot standing upright on two legs, full body, front view, round fluffy turquoise-teal fur, flat cream face mask with a tiny nose and a small cat-like smile, big round dark navy eyes with white highlights, cream inner ears pointing sideways, two tall twisted corkscrew spiral saffron-orange horns, small beard tuft, cream round belly, little saffron hooves, thick dark navy outline, solid flat colours, plain white background
```

## marko_wave_768.png and marko_wave_1024.png

768×768 and 1024×1024, same seed:

```
Marko, waving one hand. A <Marko description>
```

## marko_sign_english.png

768×768:

```
A baby markhor mascot named Marko holding up a cream sign with large bold English text written on it: "READ URDU WITH MARKO". full body, front view, round fluffy turquoise-teal fur, cream face, big navy eyes, two tall saffron corkscrew horns, thick dark navy outline, solid flat colours, plain white background
```

## classroom_welcome.png

768×768, seed 7:

```
A cozy village school classroom with a chalkboard that says "WELCOME", flat vector illustration, warm colours
```

## marko_urdu_sign.png

768×768. Made during testing with the same settings as above, before the final notebook, using the longer negative prompt:

```
A baby markhor mountain goat mascot named Marko holding up a cream sign with large Urdu text written on it: "اردو پڑھیں". full body, front view, round fluffy turquoise-teal fur, flat cream face mask with a tiny nose and a small cat-like smile, big round dark navy eyes with white highlights, cream inner ears pointing sideways, two tall twisted corkscrew spiral saffron-orange horns, small beard tuft, cream round belly, little saffron hooves, thick dark navy outline, solid flat colours, plain white background
```

The sign came out reading roughly `اردو پزخچں`: the first word is right and the second is wrong.

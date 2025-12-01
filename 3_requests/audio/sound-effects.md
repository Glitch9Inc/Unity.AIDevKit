---
icon: volume
---

# Sound Effects

Generate sound effects from text descriptions using `.GENSoundEffect()`.

## Basic Usage

```csharp
AudioClip explosion = await "Massive explosion sound"
    .GENSoundEffect()
    .ExecuteAsync();

audioSource.clip = explosion;
audioSource.Play();
```

## Input Types

### String Input

```csharp
AudioClip sfx = await "Dog barking"
    .GENSoundEffect()
    .ExecuteAsync();
```

### Prompt Input

```csharp
var prompt = new Prompt("Sound of {action} on {surface}");
AudioClip sfx = await prompt
    .GENSoundEffect()
    .ExecuteAsync();
```

## Configuration

### Duration

```csharp
AudioClip sfx = await "Thunder strike"
    .GENSoundEffect()
    .SetDuration(2.5f)  // 2.5 seconds
    .ExecuteAsync();
```

### Prompt Influence

Control how closely the AI follows your prompt (0.0-1.0):

```csharp
AudioClip sfx = await "Sci-fi laser beam"
    .GENSoundEffect()
    .SetPromptInfluence(0.8f)  // Higher = stricter adherence
    .ExecuteAsync();
```

## Unity Integration Examples

### Example 1: Dynamic SFX System

```csharp
public class DynamicSFXManager : MonoBehaviour
{
    private Dictionary<string, AudioClip> sfxCache = new();
    
    public async UniTask<AudioClip> GetSFX(string description)
    {
        // Check cache first
        if (sfxCache.ContainsKey(description))
            return sfxCache[description];
        
        // Generate new SFX
        AudioClip sfx = await description
            .GENSoundEffect()
            .ExecuteAsync();
        
        // Cache for reuse
        sfxCache[description] = sfx;
        return sfx;
    }
    
    public async UniTask PlaySFX(string description, Vector3 position)
    {
        AudioClip sfx = await GetSFX(description);
        AudioSource.PlayClipAtPoint(sfx, position);
    }
}

// Usage
await sfxManager.PlaySFX("Sword slash", transform.position);
await sfxManager.PlaySFX("Door creak", doorPosition);
```

### Example 2: Procedural Weapon Sounds

```csharp
public class WeaponSoundGenerator : MonoBehaviour
{
    public async UniTask<AudioClip> GenerateWeaponSound(string weaponType)
    {
        string description = weaponType switch
        {
            "laser" => "High-pitched sci-fi laser beam with echo",
            "shotgun" => "Powerful shotgun blast with reload",
            "sword" => "Sharp metallic sword swing through air",
            "magic" => "Magical spell casting with sparkles",
            _ => $"{weaponType} weapon sound"
        };
        
        return await description
            .GENSoundEffect()
            .SetDuration(1.5f)
            .ExecuteAsync();
    }
}
```

### Example 3: Footstep System

```csharp
public class FootstepSoundGenerator : MonoBehaviour
{
    private Dictionary<string, AudioClip> footstepCache = new();
    
    public async UniTask<AudioClip> GetFootstepSound(string surfaceType)
    {
        if (footstepCache.ContainsKey(surfaceType))
            return footstepCache[surfaceType];
        
        string description = $"Footsteps on {surfaceType}";
        AudioClip footsteps = await description
            .GENSoundEffect()
            .SetDuration(0.5f)
            .ExecuteAsync();
        
        footstepCache[surfaceType] = footsteps;
        return footsteps;
    }
}

// Usage
AudioClip woodSteps = await footsteps.GetFootstepSound("wooden floor");
AudioClip gravelSteps = await footsteps.GetFootstepSound("gravel");
AudioClip metalSteps = await footsteps.GetFootstepSound("metal grating");
```

### Example 4: Environmental Ambience

```csharp
public class AmbienceGenerator : MonoBehaviour
{
    public async UniTask GenerateAmbience(string environment)
    {
        AudioClip ambience = await $"Ambient sounds of {environment}"
            .GENSoundEffect()
            .SetDuration(10f)  // Longer duration for ambience
            .ExecuteAsync();
        
        AudioSource ambienceSource = gameObject.AddComponent<AudioSource>();
        ambienceSource.clip = ambience;
        ambienceSource.loop = true;
        ambienceSource.volume = 0.3f;
        ambienceSource.Play();
    }
}

// Usage
await ambience.GenerateAmbience("peaceful forest with birds");
await ambience.GenerateAmbience("busy city street");
await ambience.GenerateAmbience("underwater cave");
```

### Example 5: UI Sound Effects

```csharp
public class UISoundGenerator : MonoBehaviour
{
    private Dictionary<string, AudioClip> uiSounds = new();
    
    async void Start()
    {
        // Pre-generate common UI sounds
        uiSounds["click"] = await "Soft button click".GENSoundEffect().ExecuteAsync();
        uiSounds["hover"] = await "Subtle hover sound".GENSoundEffect().ExecuteAsync();
        uiSounds["success"] = await "Success chime".GENSoundEffect().ExecuteAsync();
        uiSounds["error"] = await "Error buzz".GENSoundEffect().ExecuteAsync();
    }
    
    public void PlayUISound(string soundType)
    {
        if (uiSounds.ContainsKey(soundType))
        {
            AudioSource.PlayClipAtPoint(uiSounds[soundType], Camera.main.transform.position);
        }
    }
}
```

### Example 6: Combat Sound System

```csharp
public class CombatSoundSystem : MonoBehaviour
{
    public async UniTask PlayHitSound(string weaponType, string targetMaterial)
    {
        string description = $"{weaponType} hitting {targetMaterial}";
        
        AudioClip hitSound = await description
            .GENSoundEffect()
            .SetDuration(0.8f)
            .SetPromptInfluence(0.9f)  // Precise sound
            .ExecuteAsync();
        
        AudioSource.PlayClipAtPoint(hitSound, transform.position);
    }
}

// Usage
await combatSound.PlayHitSound("sword", "metal shield");
await combatSound.PlayHitSound("hammer", "wooden door");
await combatSound.PlayHitSound("fist", "leather armor");
```

## Prompt Engineering Tips

### ✅ Good Prompts

```csharp
// ✅ Specific and detailed
"Heavy metal door slamming shut with echo"

// ✅ Include acoustic properties
"Crisp footsteps on dry leaves with rustling"

// ✅ Describe timing and dynamics
"Quick sword swoosh followed by metallic clang"

// ✅ Include environment context
"Distant thunder rumble in a rainy forest"
```

### ❌ Bad Prompts

```csharp
// ❌ Too vague
"sound"

// ❌ Too complex/multiple sounds
"explosion then footsteps then door opening then music"

// ❌ Unrealistic expectations
"sound of colors changing"

// ❌ Non-audio descriptions
"feeling of happiness"
```

## Provider Support

### ElevenLabs

```csharp
AudioClip sfx = await "Thunder strike"
    .GENSoundEffect()
    .SetDuration(2.0f)
    .SetPromptInfluence(0.8f)
    .ExecuteAsync();
```

**Features:**

- ✅ High-quality sound effects
- ✅ Duration control (0.5s - 22s)
- ✅ Prompt influence control
- ✅ Various acoustic styles

**Note:** Currently, ElevenLabs is the primary provider for sound effect generation.

## Best Practices

### ✅ Good Practices

```csharp
// ✅ Cache frequently used sounds
Dictionary<string, AudioClip> cache = new();

// ✅ Use appropriate durations
await "Quick click".GENSoundEffect().SetDuration(0.3f).ExecuteAsync();
await "Long ambience".GENSoundEffect().SetDuration(10f).ExecuteAsync();

// ✅ Pre-generate critical sounds
async void Start()
{
    explosionSound = await "Explosion".GENSoundEffect().ExecuteAsync();
}

// ✅ Clean up unused audio clips
Destroy(audioClip);
```

### ❌ Bad Practices

```csharp
// ❌ Don't generate in Update()
void Update()
{
    await "sound".GENSoundEffect().ExecuteAsync();  // NO!
}

// ❌ Don't generate very long sounds
await "ambient".GENSoundEffect().SetDuration(300f).ExecuteAsync();  // Too long

// ❌ Don't forget to clean up
// Memory leak if clips aren't destroyed

// ❌ Don't block main thread
AudioClip clip = "sound".GENSoundEffect().ExecuteAsync().Result;  // Blocks!
```

## Limitations

1. **Duration Limits**: Typically 0.5s - 22s depending on provider
2. **Quality**: May not match professional sound design
3. **Consistency**: Same prompt may produce variations
4. **Cost**: Each generation counts toward API usage

## Performance Tips

```csharp
// ✅ Good - pre-generate and cache
async void Awake()
{
    sfxCache["explosion"] = await "Explosion".GENSoundEffect().ExecuteAsync();
    sfxCache["jump"] = await "Jump sound".GENSoundEffect().ExecuteAsync();
}

// ✅ Good - parallel generation
var tasks = new[]
{
    "Sound 1".GENSoundEffect().ExecuteAsync(),
    "Sound 2".GENSoundEffect().ExecuteAsync()
};
await UniTask.WhenAll(tasks);

// ❌ Bad - sequential generation
for (int i = 0; i < 10; i++)
{
    await $"Sound {i}".GENSoundEffect().ExecuteAsync();  // Slow!
}
```

## Error Handling

```csharp
try
{
    AudioClip sfx = await "Thunder"
        .GENSoundEffect()
        .ExecuteAsync();
    
    if (sfx == null || sfx.length == 0)
        throw new Exception("Invalid audio clip");
    
    AudioSource.PlayClipAtPoint(sfx, transform.position);
}
catch (AIApiException ex)
{
    Debug.LogError($"SFX generation failed: {ex.Message}");
}
catch (Exception ex)
{
    Debug.LogError($"Unexpected error: {ex.Message}");
}
```

## Common Use Cases

| Use Case | Example Prompt |
|----------|---------------|
| **Footsteps** | "Footsteps on gravel" |
| **Weapons** | "Sword slash through air" |
| **Impacts** | "Heavy object hitting wood" |
| **UI** | "Soft button click" |
| **Ambience** | "Forest ambience with birds" |
| **Magic** | "Magical spell casting sound" |
| **Vehicles** | "Car engine revving" |
| **Weather** | "Heavy rain on roof" |

## Next Steps

- [Voice Change](voice-change.md) - Modify voice characteristics
- [Audio Isolation](audio-isolation.md) - Clean up audio
- [Text to Speech](text-to-speech.md) - Generate speech

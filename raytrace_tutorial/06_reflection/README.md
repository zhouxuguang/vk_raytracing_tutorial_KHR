# 06 Reflection - Tutorial
![](/docs/images/06.png)

This tutorial demonstrates how to implement realistic reflections in ray tracing by tracing secondary rays when light bounces off surfaces. It introduces two different approaches: recursive reflection (using hardware recursion) and iterative reflection (using explicit loops), showing the trade-offs between simplicity and scalability.

## Key Changes from 02_basic.cpp

### 1. Shader Structure and Reflection Modes

**Modified: `shaders/rtreflection.slang`**
The shader supports two reflection implementations controlled by preprocessor defines:
```hlsl
// #define REFLECTION_RECURSIVE  // Hardware recursion approach
#define REFLECTION_ITERATIVE    // Loop-based approach
```

### 2. Extended Payload Structure

**Modified: `shaders/rtreflection.slang`**
The hit payload is extended to track reflection state and accumulate results:
```hlsl
struct HitPayload
{
    float3 color;      // Final color for this ray
    float  weight;     // Reflection weight (decreases with each bounce)
    int    depth;      // Current reflection depth
    
    // For iterative reflection
    float3 rayOrigin;    // Origin for next ray iteration
    float3 rayDirection; // Direction for next ray iteration
};
```

### 3. Push Constants

**Modified: `shaders/shaderio.h`**
Added depth control for reflection quality:
```cpp
struct TutoPushConstant
{
    // ... existing members ...
    int depthMax = 3;  // Maximum reflection depth (default: 3)
};
```

### 4. Ray Generation Shader - Iterative Approach

**Modified: `shaders/rtreflection.slang`**
The iterative approach uses a loop in the ray generation shader to handle multiple bounces:
```hlsl
// Iterative reflection loop
while(payload.depth < pushConst.depthMax && payload.weight > 0.01)
{
    float prevWeight = payload.weight;
    TraceRay(topLevelAS, rayFlags, 0xff, 0, 0, 0, ray, payload);
    
    accumulatedColor += payload.color * prevWeight;
    
    // Update ray for next iteration
    ray.Origin = payload.rayOrigin;
    ray.Direction = payload.rayDirection;
}
```

### 5. Closest Hit Shader - Reflection Logic

**Modified: `shaders/rtreflection.slang`**
Both approaches calculate reflection direction and handle material properties:
```hlsl
// Calculate reflection direction
float3 reflectionDir = reflect(-V, N);

// For recursive: create new ray and trace recursively
// For iterative: store ray info in payload for next iteration
payload.rayOrigin = worldPos;
payload.rayDirection = reflectionDir;
payload.weight *= metallic;  // Decrease weight based on material
```

### 6. C++ Application Changes

**Modified: `06_reflection.cpp`**

#### Pipeline Configuration
Increased recursion depth to support multiple reflection bounces:
```cpp
#define MAX_DEPTH 10U
rtPipelineInfo.maxPipelineRayRecursionDepth = std::max(MAX_DEPTH, m_rtProperties.maxRayRecursionDepth);
```

#### UI Controls
Added reflection depth slider for quality control:
```cpp
PE::SliderInt("Reflection Depth", &m_pushValues.depthMax, 1, MAX_DEPTH, "%d", 
              ImGuiSliderFlags_AlwaysClamp, "Maximum reflection depth");
```

### 7. Scene Setup

**Modified: `createScene()`**
- Uses `wuson.glb` model with metallic materials for better reflection demonstration
- Configures mirror surfaces and sky environment for visible reflections
- Sets up directional lighting to create shadows and highlights

## How It Works

### Recursive Approach
The recursive method uses hardware ray recursion where each hit shader can spawn new rays:
1. **Primary Ray**: Hits surface and calculates direct lighting
2. **Reflection Check**: If material is metallic, calculates reflection direction
3. **Recursive Call**: `TraceRay()` is called recursively from within the closest hit shader
4. **Depth Control**: Stops when depth limit is reached or reflection weight becomes negligible
5. **Hardware Limitation**: Limited by GPU's `maxRayRecursionDepth` (typically 4-8 levels)

### Iterative Approach
The iterative method uses explicit loops in the ray generation shader:
1. **Primary Ray**: Hits surface and calculates direct lighting
2. **Payload Update**: Reflection information is stored in payload for next iteration
3. **Loop Control**: Ray generation shader continues until depth or weight limits
4. **Accumulation**: Each iteration's color is accumulated with previous results
5. **No Hardware Limits**: Can handle arbitrarily deep reflection chains

## Benefits

### Iterative Over Recursive
- **No Hardware Limits**: Avoids GPU recursion depth limitations
- **Better Memory Management**: Uses constant memory regardless of reflection depth
- **Predictable Performance**: Linear performance scaling with depth
- **Easier Debugging**: Linear execution flow is easier to trace
- **Scalability**: Can handle deep reflection chains for complex scenes

### General Benefits
- **Realistic Lighting**: Simulates light bounces for photorealistic rendering
- **Material-Based**: Different materials reflect differently based on metallic/roughness properties
- **Progressive Quality**: More reflection bounces create more realistic lighting
- **Performance Control**: Can limit depth to balance quality vs performance

## Technical Details

### Reflection Quality Control
- **Depth Limit**: `depthMax` parameter controls maximum bounces (1-10, default: 3)
- **Weight Decay**: Reflection contribution decreases with each bounce based on material metallic value
- **Early Termination**: Stops when weight becomes too small (< 0.01) to avoid unnecessary computation
- **Material Threshold**: Only materials with metallic > 0.01 generate reflections

### Performance Considerations
- **Exponential Cost**: Each reflection level increases computation exponentially
- **Memory Usage**: Iterative mode uses more payload data but avoids stack overflow
- **Material Optimization**: Early termination based on material properties
- **Hardware Differences**: Recursive depth limits vary between GPU vendors

### Material Properties
- **Metallic**: Controls reflection strength (0 = dielectric, 1 = metallic)
- **Roughness**: Controls reflection scatter (0 = mirror, 1 = diffuse)
- **Weight Decay**: Reflection contribution decreases with each bounce

## Usage Instructions

- **Reflection Depth**: Use the UI slider to adjust quality (1-10 bounces)
- **Material Setup**: Ensure models have proper metallic/roughness values for visible reflections
- **Performance Tuning**: Lower depth for real-time applications, higher for offline rendering
- **Scene Design**: Include reflective surfaces and interesting geometry for best visual results

## Next Steps

This reflection technique can be extended to:
- **Refraction**: Add transmission through transparent materials
- **Glossy Reflections**: Implement roughness-based reflection blur
- **Fresnel Effects**: Add realistic reflection based on viewing angle
- **Global Illumination**: Combine with other lighting effects for complete lighting simulation

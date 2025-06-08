Postprocessing in NGE is anything that happens after the scene has been rendered but before it is displayed on the screen. This includes effects like bloom, depth of field,  and more.

Postprocessing effects can be attached to a viewport with a component that implements [ViewPortFragment](https://javadoc.ngengine.org/org/ngengine/components/fragments/ViewPortFragment.html)  and overrides the `loadViewPortFilterPostprocessor` method, from there you can add your desired postprocessing effects to the [FilterPostProcessor](https://javadoc.ngengine.org/com/jme3/post/FilterPostProcessor.html) instance that is passed preconfigured to the method.

!!! example
    ```java
    public class MyPostProcessingComponent implements Component<Object>, ViewPortFragment {
        @Override
        public void loadViewPortFilterPostprocessor(AssetManager assetManager, FilterPostProcessor fpp){
            // add your postprocessing effects here
            // fpp.addFilter(new BloomFilter());
            // ....
        }

        @Override
        public void updateViewPort(ViewPort viewPort,float tpf) {}
    }
    ```


!!! tip
    The current version of NGE inherits its postprocessing capabilities from jMonkeyEngine (jME3), you can refer to the jME3 wiki for more info [jME3 Postprocessing Documentation](https://wiki.jmonkeyengine.org/docs/3.4/core/effect/effects_overview.html).


## ToneMap

!!! abstract "See [ToneMapFilter Javadoc](https://javadoc.ngengine.org/com/jme3/post/filters/ToneMapFilter.html)"

Tone mapping is a postprocessing effect that adjusts the brightness and contrast of a scene to simulate the way cameras capture light. It helps to create a more balanced and visually appealing image, especially in high dynamic range (HDR) scenes.

```java
ToneMapFilter toneMap = new ToneMapFilter();
toneMap.setWhitePoint(1.0f);
fpp.addFilter(toneMap);
```


## Bloom

!!! abstract "See [BloomFilter Javadoc](https://javadoc.ngengine.org/com/jme3/post/filters/BloomFilter.html)"

Bloom is a postprocessing effect that simulates the way bright light sources bleed into surrounding areas, creating a glowing effect. It enhances the visual appeal of scenes with bright lights.

```java
BloomFilter bloom=new BloomFilter();
bloom.setDownSamplingFactor(2);
bloom.setBlurScale(1.37f);
bloom.setExposurePower(3.30f);
bloom.setExposureCutOff(0.2f);
bloom.setBloomIntensity(2.45f);
fpp.addFilter(bloom);
```

## Depth of Field

!!! abstract "See [DepthOfFieldFilter Javadoc](https://javadoc.ngengine.org/com/jme3/post/filters/DepthOfFieldFilter.html)"

Depth of Field (DoF)  is a postprocessing effect that simulates the way cameras focus on objects at different distances, blurring out-of-focus areas. It creates a more realistic and cinematic look.

```java
DepthOfFieldFilter dof = new DepthOfFieldFilter();
dof.setFocusDistance(10f);
dof.setFocusRange(5f);
dof.setBlurScale(2f);
fpp.addFilter(dof);
```


## Light Scattering

!!!abstract "See [LightScatteringFilter Javadoc](https://javadoc.ngengine.org/com/jme3/post/filters/LightScatteringFilter.html)"

Light scattering simulates volumetric light at a given position


```java
LightScatteringFilter filter = new LightScatteringFilter(lightPos);
fpp.addFilter(filter);
```

## Screen Space Ambient Occlusion (SSAO)

!!! abstract "See [SSAOFilter Javadoc](https://javadoc.ngengine.org/com/jme3/post/ssao/SSAOFilter.html)"

Screen Space Ambient Occlusion (SSAO) is a postprocessing effect that simulates the way light interacts with surfaces, creating soft shadows in crevices and corners. It enhances depth perception and realism in scenes.

```java
SSAOFilter ssaoFilter = new SSAOFilter(2.9299974f,32.920483f,5.8100376f,0.091000035f);
ssaoFilter.setApproximateNormals(true);
fpp.addFilter(ssaoFilter);
```

## Fog
!!! abstract "See [FogFilter Javadoc](https://javadoc.ngengine.org/com/jme3/post/filters/FogFilter.html)"

Fog is a postprocessing effect that adds a layer of fog to the scene, it can be used to create atmospheric effects or to hide distant objects. It can be configured to use different colors and densities.

```java
FogFilter fog = new FogFilter();
fog.setFogColor(ColorRGBA.Gray);
fog.setFogDistance(100f);
fpp.addFilter(fog);
```


## Shadows

!!! abstract "See [PointLightShadowFilter Javadoc](https://javadoc.ngengine.org/com/jme3/shadow/PointLightShadowFilter.html)"
!!! abstract "See [DirectionalLightShadowFilter Javadoc](https://javadoc.ngengine.org/com/jme3/shadow/DirectionalLightShadowFilter.html)"
!!! abstract "See [SpotLightShadowFilter Javadoc](https://javadoc.ngengine.org/com/jme3/shadow/SpotLightShadowFilter.html)"


Shadows in NGE are rendered as post pcrocessing effects. There is one Filter for each type of light source, that can be added to the `FilterPostProcessor` instance to render its shadow

| light source class | shadow filter class |
|--------------------|---------------------|
| PointLight         | PointLightShadowFilter |
| DirectionalLight   | DirectionalLightShadowFilter |
| SpotLight          | SpotLightShadowFilter |
| AmbientLight       | (not applicable) |



Shadow calculations (cast and receive) have a performance impact, so use them sparingly. You can turn off shadow casting for individual geometries, for portions of the scene graph, or for the entire scene:

```java
spatial.setShadowMode(ShadowMode.Inherit);     // This is the default setting for new spatials.
rootNode.setShadowMode(ShadowMode.Off);        // Disable shadows for the whole scene, except where overridden.
airplane.setShadowMode(ShadowMode.Cast);       // There's nothing above the airplane to cast shadows on it.
ghost.setShadowMode(ShadowMode.Off);           // The ghost is translucent: it neither casts nor receives shadows.
```


## More

For more postprocessing effects, refer to the [Javadoc](https://javadoc.ngengine.org/com/jme3/post/filters/package-summary.html) or the [jME3 wiki](https://wiki.jmonkeyengine.org/docs/3.4/core/effect/effects_overview.html).

!!! tip 
    You can also create your own custom postprocessing effects by extending the `Filter` class and implementing the necessary methods. They are screen-space shader effects, not dissimilar to the way materials work, but they are applied to a quad covering the entire screen rather than to individual geometries.
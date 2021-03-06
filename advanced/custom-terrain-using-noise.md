---
description: Making your terrain do crazy stuff; masters only
---

# Custom terrain using noise

Mostly all terrain generated in EpicWorldGenerator is generated using gradient noise. Here is what Wikipedia says about gradient noise;

{% hint style="info" %}
> Gradient noise is a type of noise commonly used as a procedural texture primitive in computer graphics. It is conceptually different, and often confused with value noise. This method consists of a creation of a lattice of random \(or typically pseudo-random\) gradients, which are then interpolated to obtain values in between the lattices. An artifact of some implementations of this noise is that the returned value at the lattice points is 0. Unlike the value noise, gradient noise has more energy in the high frequencies.
{% endhint %}

The most common know implementation of gradient noise is Perlin and open simplex noise. The noise type that the plugin will use is depending on the options that have been set in the `world-settings.json`. When navigating to your `world-settings.json` file in your `*/<world name>/settings/`, you will find the following line written in the file.

```javascript
"useOpenSimplexNoise": false,
```

If `"useOpenSimplexNoise"` is set to false then that gives access to the plugin to use Perlin as the default terrain noise generator for the plugin. Read below to understand more about both Simplex and Perlin noise generation.

Perlin is for servers who have more ram, and can use more CPU. Although both Open Simplex and Perlin are much alike, Open Simplex is recommended for those who don't have a server with a lot of ram or CPU usage. Perlin does create better quality terrain, however, uses more performance than open simplex.

#### How does Perlin noise look like?

Well, it simply look something like this;  
![render-noise.png](http://www.redblobgames.com/maps/terrain-from-noise/images/render-noise.png)

#### Creating terrain out of the noise

To simplify the process will we now start to look into 1 dimensional noise instead of 2D.

Here is the processes;  
We start with one layer. We then create a few more layers with higher frequency. We also decrease the amplitude of the next layers. This lets us create a custom terrain that looks cool and does its job.  
![](http://i.imgur.com/4OWFZ84.png)

If we put all the layers on top of each other, we get a terrain looking like this;  
![](http://i.imgur.com/vl8ydoj.png)

This noise will be stacked on top of the existing terrain. Without noise will the plugin generate a landscape at the height of the `baseheight` property that can be found in the biome. Here is illustration of how it looks like in EWG;  
![](http://i.imgur.com/LZctDD6.png)

The green line indicate the base height. The blue indicate the water level. The black line that is wave is the main noise layer. The gray wave is the next layer, that has higher frequency and lower scale. The height in the world is here;

* the baseheight  
* main layer   
* secondary noise layer.  

  = Terrain height

The `baseheight` property is explained more below.

Make sure to check out the "Frequency" and "Octaves" section at [http://www.redblobgames.com/maps/terrain-from-noise/](http://www.redblobgames.com/maps/terrain-from-noise/)

After figuring out how maps are generated by sounds, and the different variation they use, we can move to making our own custom terrain based on these guidelines and principles.

First of all, lets implement the data we need for custom terrain before we get started into configuring it.

1. Open your `<biome>.json` `*/<world name>/settings/biomes/custom` or `biomes/default` based on what you did.
2. You can paste in the configuration below based on `.json` syntax in order to not throw any errors. A guide is provided below with one already pasted in, so you don't rip your head off while doing this.
3. If you are using the guide, don't use anything else other than the part about the custom terrain since everything else in there \(as well as the custom terrain, but we're editing that\) is just default configuration.
4. Locate terrain settings in your `<biome>.json` file. Change the provider name \(text inside quotes\) to `"CUSTOM_EWG"` the name must be all capitals \(don't include the quotes, they are provided in your `.json` file\).
5. You're done, now lets customize.

Guide: [http://jetpad.io/index.php?id=5153513](http://jetpad.io/index.php?id=5153513)

#### Implement your terrain

```javascript
  "customTerrain": {
    "noiseLayers": [ //List of layers, or "octaves"
      {//Base layer (80 blocks heigh)
        "xDensity": 300.0, //Inverted frequency in the x axis. (x-cord/xDensity). Increasing this number create lower frequency which creates more mountains and higher ranges. Also, the xDensity stretches the world east and west, meaning, the higher the number is, the more it stretches east or west. Setting this to around 150-200 is your best choice.
        "yDensity": 300.0,//Inverted frequency in the y axis. (z-cord/yDensity). Increasing this number create lower frequency. Take a huge note here, y and z is mixed in minecraft.  Increasing this number creates lower frequency which creates higher mountains and higher ranges. The yDensity stretches the world north and south, meaning the higher the number is, the more it stretches north and south. Keeping this at the SAME NUMBER OF your xDensity is best.
        "multiplier": {//The noise is a value from -1 to 1, we need to make it a value that fits our terrain. We already have a baseHeight defined in the setting files, often around 60.
          "baseMultiplier": 80.0, //Makes the layer peak at y level = 80. Gives the average base height of the terrain. given a base height at 60 the peak height will be 140. It's best for this number to be around 60-70
          "biomeModifier": 0.0, //Percentage of how much the layer should care about the biome given in a scale where 1 = 100%, more than 100% is allowed. This means if it is set to 1, the layer height will return 0 in the edges of the biome. NOT REQUIRED TO BE TOUCHED.
          "riverModifier": 0.0,//Percentage of how much the layer should care about the rivers given in a scale where 1 = 100%, more than 100% is allowed. This means if it is set to 1, the layer height will return 0 near the rivers in the biome. NOT REQUIRED TO BE TOUCHED.
          "baseModifiers": [ //Might be a direction modifier. In this example, it will make the waves lower when they are negative using a multiplier. NOT REQUIRED TO BE TOUCHED.
            {
              "referenceLayer": -1,//The layer height the start and stop refer to. In this case, the total layer height. Best to refer to -1.
              "start": 1.0E-9, //When it should start fading to a value
              "stop": 0.0, ///When it should stop
              "finalCoefficient": 0.2857143 //Final multiplier. In this case, 1/3.5
            }
          ]//Adding multiple modifiers can modify the wave in the directions you want, if you add a value higher than 1
        }
      },
      {//Detail layer. These layers are added in order to make your custom terrain more believable and playable. You can add ass many layers as you would like, but there are 3 here as templates. Make sure you change everything and to make sure that each one of these is changed to a different setting. You will have to play with all these settings in order to find a biome that's the perfect match for you, and your players.(5 blocks heigh)
        "xDensity": 20.0,
        "yDensity": 20.0,
        "multiplier": {
          "baseMultiplier": 5.0,
          "biomeModifier": 0.0,
          "riverModifier": 0.0,
          "baseModifiers": [
            {
              "referenceLayer": -1,
              "start": 35.0,
              "stop": 0.0,
              "finalCoefficient": 0.2
            }
          ]
        }
      },
      {//Detail layer (3 blocks heigh)
        "xDensity": 12.0,
        "yDensity": 12.0,
        "multiplier": {
          "baseMultiplier": 3.0,
          "biomeModifier": 0.0,
          "riverModifier": 0.0,
          "baseModifiers": [
            {
              "referenceLayer": -1,
              "start": 35.0,
              "stop": 0.0,
              "finalCoefficient": 0.2
            }
          ]
        }
      },
      {//Detail layer (1.5 blocks heigh)
        "xDensity": 5.0,
        "yDensity": 5.0,
        "multiplier": {
          "baseMultiplier": 1.5,
          "biomeModifier": 0.0,
          "riverModifier": 0.0,
          "baseModifiers": [
            {
              "referenceLayer": -1,
              "start": 35.0,
              "stop": 0.0,
              "finalCoefficient": 0.1
            }
          ]
        }
      }
    ]
  },
```

When trying to copy these settings, copy [this](http://raw.jetpad.io/index.php?id=30a32c0). If you copy the ones above, the plugin will throw errors.

This configuration is by far the most complicated one in EpicWorldGenerator. Though we try to explain things as clearly as possible, some things may not be able to come clearly to you as it did with us. We have gave recommended numbers for several parts during the configuration, please use those if you do not know what to do. Good luck with your own terrain!

This configuration requires much patience, and should be carefully configured. If you are doing this for the first time, you will have to go through different combinations of settings to find what works for you. Although many things will be challenging, please keep trying as that is the reason why this feature was implemented.

#### Support

Please try to resolve this yourself, as even the most experienced among our support have serious trouble doing this themselves, let alone explain it.

If you don't care, and have any issues with any of the settings in this file, make sure to contact our support team at [Discord](https://discord.gg/Jq3ecb3).

**Back to:** [**Table of contents**](https://docs.dynamic-bytes.com/table-of-contents)**.**


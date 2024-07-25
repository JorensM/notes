Pixel Plant dev

## Tick system

Implement a tick system to improve performance. Basically instead of updating plants every draw loop, update them every 'tick'. A tick can be a defined number of seconds, for example 1 second or 5 seconds. This will improve the performance.

## Auto save

Plant data should be saved to disk every tick or every few ticks. Also current time should be saved (see next section)

## Calculating water levels on game launch

Every time the game is launched, the time that has passed since last launch should get calculated and the respective amount should be applied to water level & growth. To calculate time that has passed, subtract the last saved time from current time.

To calculate the new water levels multiply the time passed by the water level decrease rate.

To calculate growth:

```
w1 = water level when player opened game
w2 = water level when player closed the game
g = 0 (growth to be calculated)
for each water stage:
     r = rate of growth at this water stage
     o = amount of overlap between [water_stage_range] and [w1, w2]
     g += r * o
endfor     
```

## WaterLevel class

Tracks water level and specifies stages of water level and growth speeds for those levels

Variables
  * int current_level
  * int decrease_rate - how much level decreased per tick
  * array[vector2]stages

Methods
  * decrease(ticks) - decrease water level according to amount of ticks provided
  * decrease() - decrease water level by 1 tick

## Plant class
Tracks information about a plant

Variables
  * growth_per_tick - how much the plant grows per tick
  * growth - how much the plant has growb
  * plant_time - when the plant was planted
  * WaterLevel water_level

Methods
  * grow(ticks) - grow plant by n ticks. Should keep in mind water level stages
  * grow() - grow by 1 tick
  * water() - refill water level

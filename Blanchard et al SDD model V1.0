extensions [rngs]
breed [socials social]                 ;all of these fish will want high conspecific densities
breed [heterogeneouss heterogeneous]   ;this population will want a wide range of conspecific densities
breed [asocials asocial]               ;all of these fish will want low conspecific densities
breed [e_asocials e_asocial]           ;these are extremely asocial fish
breed [e_socials e_social]             ;these are extremely social fish
breed [Controls Control]               ;these fish lack any behavioral information
breed [Nulls null]                     ;these fish have total random personalities with equal probability of every value between 0 and 1
breed [Bimodals bimodal]               ;creates a population where half are social and half are asocial
turtles-own
[
  SOCIABILITY                          ;state variable bound between 0 and 1
  TARGET_DENSITY                       ;dependent on sociability
  MY_DENSITY                           ;full knowledge of local conspecific density
  SOCIAL_RANK                          ;dependent on sociability: bottom up ranked sociability
  MAX_SEARCH_RADIUS                    ;dependent on Max_moves_per_tick
  PANIC_TIME                           ;dependent on dispersal window (10%)
  TIME_SINCE_LAST_POOL_FOUND           ;counter which tracks how many days have elapsed since the fish last found a pool
  DAYS_UNHAPPY                         ;counter which tracks how many days have elapsed since the fish has encountered a suitable density
  LAST_POOL_FOUND                      ;patch identifier: coordinates of the last pool the fish evaluated
  BEEN_HERE_BEFORE?                    ; boolean identifier set to true if the fish has been in that pool before and is evaluating it's density
  FIRST_PATCH                          ;patch identifier: coordinates the fish started at
  LAST_PATCH                           ;patch identifier: coordinates the fish was last at
  TOTAL_DISTANCE_TRAVELED              ;cummulative distance traveled from last-patch to current coordinates, added on every tick
  LINEAR_DISTANCE_TRAVELED             ;distance from current patch to first patch on the final tick
  TIME_SPENT_HERE                      ;counter which tracks how many days a turtle has been in a pool
  STAY?                                ;boolean identifier: set to true if the fish is either happy or waiting for conditions to improve in the pool
  HANG_OUT_COUNTER                     ;dependent on sociability: counts down by 10% every day the fish is unhappy
]

patches-own
[
  DEPTH                                ;state variable
  DENSITY                              ;density of fish at the start of the day
  MEAN_SOCIABILITY                     ;mean sociability of turtles on the patch, called by sociability list
  VARIANCE_LIST                        ;variance of sociability turtles in the hole
  STDEV_LIST                           ;standard deviation of sociability turtles in the hole
]

globals
[
  ALPHA                                ;beta distribution parameter
  BETA                                 ;beta distribution parameter
  DIST_COUNT                           ;count of sociability distributions activated for error checking purposes
  RANDOM_FISH                          ;a random currently living fish
  START#                               ;the number of fish at day 1
  CURRENT_COUNT                        ; the current number of living fish
  SOCIAL_RANK_LIST                     ;social ranking (lowest to highest sociability scores)
  DISTANCE_LIST                        ;list of sociabilities and associated total distances traveled
                                       ;DISPERSAL_WINDOW                    ;;DEFINED ON INTERFACE= total run time
                                       ;MAX_TARGET                          ;;DEFINED ON INTERFACE= maximum preferred density if sociability = 1
                                       ;MAX_MOVES_PER_TICK                  ;;DEFINED ON INTERFACE= maximum linear movement distance of a single fish in a single da+y
                                       ;START_FISH_COUNT                    ;;DEFINED ON INTERFACE= initial population size to be generated
  CUT_RUN_SHORT?
  DENSITY_LIST                         ;LIST OF X COORDINATE AND ASSOCIATED DENSITIES and densities
  SOCIABILITY_LIST                     ;x coordinate, y coordinate and mean sociability of turtles in the patch
]


to setup
  clear-all
  reset-ticks
  stop-inspecting-dead-agents
  set_globals                                                               ;if using optimal settings this will set all global factors to those used in experiments
  make_pools
  make_fish
  if Show_social_v_asocial_path? = TRUE
  [ if user-yes-or-no? ["To use this option you must select only the null or hetero distributions in the panel on the left, please ensure you have done so before proceeding"]
    [
      ask one-of turtles with [sociability > 0.9] [set color blue set size 4 pen-down ask other turtles with [sociability > 0.5] [set size 0]]
      ask one-of turtles with [sociability < 0.1] [set color red set size 4 pen-down ask other turtles with [sociability < 0.6] [set size 0]]
    ]
  ]

 ; ifelse EXPERIMENT_SETTINGS? = FALSE
 ; [update_plots]                                                            ;if not running an experiment then update the plots in the window, but don't if experimenting (speed optimization)
 ; [ask turtles [set size 0]]                                                ;if experimenting, make all turtles invisible (speed optimization)

  ask patches [set DENSITY count turtles-here]
  ; ask one-of patches [set DENSITY_LIST list "," DENSITY_LIST]

  set social_rank_list sort-on [sociability] turtles                        ;sort the social rank by sociability low to high
  Look_at_one_fish                                                          ;if watch_a_random_fish? is activated this makes all other fish invisible and tracks one random fish
  set START# count turtles
  fish_initial_state                                                        ;sets all turtle owned parameters to their initial state for all fish

end

to go
  tick
  if count turtles <= 10 [stop]                                             ;lines 2 and 3 of Go are error checking protocols to stop any runs that are using undesired scenarios (experiment speed optimization)
  if DIST_COUNT > 1 [ask n-of (count turtles - 1) turtles [die]]

  ask patches [set DENSITY count turtles-here]
  ask turtles [set MY_DENSITY count turtles-here]

  if Show_social_v_asocial_path? = TRUE
  [
    ask turtles with [size > 1] [set label precision TOTAL_DISTANCE_TRAVELED 2]
  ]


  ifelse Control? = TRUE
    [ask Controls [Control_pool_search]]                                    ;if running a Control scenario, then tell Controls to search for pools without using social information
    [  ifelse SOCIAL_RANK_ordered_movement? = TRUE                          ;if you want to use social rank ordered movement, then make the list and tell the fish to search for a pool in order of that list
      [let rank-list sort-on [SOCIAL_RANK] turtles
        foreach rank-list
        [ask ? [pool_search]]]
      [ask turtles [pool_search]]                                           ;if not, then just tell the fish to search for a pool in a random order
    ]

  ask turtles [track_travel]                                                ;this keeps track of how far each fish has moved

  if ticks >= DISPERSAL_WINDOW [  Final_tick_procedures
    stop]                                    ;stops the run and updates the plots/outputs when the dispersal window closes


  if watch_a_RANDOM_FISH? = TRUE                                            ;if you're watching one fish, this block tells it to highlight it's search radius each day with a white splotch
    [ask turtle RANDOM_FISH [show MY_DENSITY ask patches in-radius MAX_SEARCH_RADIUS [set pcolor white ]]]

  if CUT_RUN_SHORT? = TRUE [Final_tick_procedures
    stop]
  update_plots
end


;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;movement protocols;;;;;;;;;;;movement protocols;;;;;;;;;;;;;movement protocols;;;;;;;movement protocols;;;;;;;;;;;;;;;;;movement protocols;;;;;;;;;;;;;;;;;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
to Control_pool_search

  let D min [DEPTH] of patches in-radius MAX_SEARCH_RADIUS                                                     ;minimum DEPTH of all patches reachable in one day
  let Dn min-one-of patches in-radius MAX_SEARCH_RADIUS [DEPTH]                                                ;patch with the lowest DEPTH which can be reached in one day
  let LDe [DEPTH] of patch-here                                                                                ;current patch DEPTH

  ifelse patch-here = LAST_POOL_FOUND                                                                          ;note if you've found the same pool that you most recently evaluated
    [set BEEN_HERE_BEFORE? TRUE ]
    [set BEEN_HERE_BEFORE? FALSE ]


  ifelse DEPTH < 0                                                                                             ;if you find a pool then reset your counter, if not then add to it
    [set TIME_SINCE_LAST_POOL_FOUND 0]
    [set TIME_SINCE_LAST_POOL_FOUND TIME_SINCE_LAST_POOL_FOUND + 1]

  ifelse DEPTH < 0                                                                                             ;if you're in a POOL
    [move-to patch-here                                                                                        ;center yourself in it
      ifelse ((random-float 101) > 50)
      [set color yellow stop]
      [set color pink new_pool_search]                                                                                                     ;stop moving
    ]

    [ifelse LDe > D AND D < 0                                                                                  ;if your current patch DEPTH is greater than the minimum DEPTH you could achieve in one day
      [move-to one-of patches in-radius MAX_SEARCH_RADIUS with-min [DEPTH]
        stop
      ]                                                                                                        ;go the the deeper patch and stop moving
      [new_pool_search]                                                                                        ;if not, then go find a pool
    ]
end

to new_pool_search
  ifelse TIME_SINCE_LAST_POOL_FOUND < PANIC_TIME                                                               ;is it time to panic about how long it has been since you found a pool?
    [right random-float SMALL_SEARCH_RADIUS                                                                    ;if not, then keep searching your local area intensively (you found a pool here before, there's probably another one nearby right?)
      forward random-float MAX_MOVES_PER_TICK
    ]
    [right random-float BIG_SEARCH_RADIUS                                                                      ;if you are panicking then start a broader search pattern (you're not having any luck here so lets expand the search area)
      forward random-float MAX_MOVES_PER_TICK
    ]
end

to pool_search

  let D min [DEPTH] of patches in-radius MAX_SEARCH_RADIUS                                                     ;minimum DEPTH of all patches reachable in one day
  let Dn min-one-of patches in-radius MAX_SEARCH_RADIUS [DEPTH]                                                ;patch with the lowest DEPTH which can be reached in one day
  let LDe [DEPTH] of patch-here                                                                                ;current patch DEPTH



  ifelse patch-here = LAST_POOL_FOUND                                                                          ;note if you're in the last pool you abandoned or not
  [set BEEN_HERE_BEFORE? TRUE ]
  [set BEEN_HERE_BEFORE? FALSE ]

  ifelse patch-here = LAST_PATCH                                                                               ;if you haven't moved today, add one to your counter. if you have, then set that counter to 0
  [set TIME_SPENT_HERE TIME_SPENT_HERE + 1]                                                                    ;this counter is used in the hang out probability calculation
  [set TIME_SPENT_HERE 0]

  ifelse DEPTH < 0                                                                                             ;if you're in a pool then set that counter to 0, if not then add a day to it
    [set TIME_SINCE_LAST_POOL_FOUND 0]
    [set TIME_SINCE_LAST_POOL_FOUND TIME_SINCE_LAST_POOL_FOUND + 1]



  ifelse DEPTH < 0                                                                                             ;if you're in a POOL
    [move-to patch-here                                                                                        ;center yourself in it
      should_I_hang_out                                                                                        ;decide if you should stay or not
      ifelse stay? = TRUE                                                                                      ;if you decide to stay then center yourself and stop for the day
      [move-to patch-here
        stop
      ]

      [set LAST_POOL_FOUND patch-here                                                                          ;if you decided to move on then note the location of the pool and search for a new one
        new_pool_search
      ]
    ]

    [ifelse LDe > D AND D < 0                                                                                  ;if your current patch DEPTH is greater than the minimum DEPTH you could achieve in one day
      [move-to Dn
        stop
      ]                                                                                                        ;go the the deeper patch and stay there for the day

      [new_pool_search]
    ]

end


to should_I_hang_out                                                                                          ;this calculates the STAY? factor as a function of time, sociability and patch suitability

  let acceptable ((DENSITY >= (TARGET_DENSITY * (1 + DENSITY_WIGGLE_ROOM))) AND DENSITY < (TARGET_DENSITY * (1 - DENSITY_WIGGLE_ROOM))) ;be happy if you're in a patch that's within your the wiggle room range of density


  if TIME_SPENT_HERE < 2[  set HANG_OUT_COUNTER (MAX_HANG_OUT_TIME * SOCIABILITY)]                            ;if you just got there, set your hang out counter to the individual's maximum possible value

  move-to patch-here                                                                                          ;center yourself in the patch

  ifelse Acceptable
  [set stay? TRUE                                                                                             ;if you're happy then stick around and reset your hang out counter
    set HANG_OUT_COUNTER (MAX_HANG_OUT_TIME * SOCIABILITY)
    stop
  ]

  [ifelse (random-float MAX_HANG_OUT_TIME) > HANG_OUT_COUNTER                                                 ;if you're not happy then draw a random number.
                                                                                                              ;if it's greater than your hang out counter
    [set stay? FALSE                                                                                          ;note the location of your pool, leave, and max out your counter again to prepare for the next time
      set LAST_POOL_FOUND patch-here

      set HANG_OUT_COUNTER (MAX_HANG_OUT_TIME * SOCIABILITY)
    ]

    [set HANG_OUT_COUNTER (HANG_OUT_COUNTER * 0.9)                                                           ;if not then deduct 10% from your counter and stay for the rest of the day
      set stay? TRUE
    ]
  ]

end





to wall_jump                                                                                                 ;this procedure makes fish turn away from world boundaries by having them draw an opposing random heading when they're within 5 patches of an edge
  if abs pxcor >= (max-pxcor - 5)
  [set heading (360 - (heading + random-float 90))]

  if abs pycor >= (max-pycor - 5)
  [set heading (270 - (heading + random-float 90))]

  if abs pxcor <= (min-pxcor + 5)
  [set heading (180 - (heading + random-float 90))]

  if abs pycor >= (min-pxcor + 5)
  [set heading (360 - (heading + random-float 360))]

end

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;world creation;;;;;;;;;;;;;world creation;;;;;;;;;;;;;world creation;;;;;;;;;;world creation;;;;;;;;;;;;;;;;;world creation;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;


to make_pools
  ifelse Grid_or_Rand_landscape? = TRUE
  [
    ask patches with [pxcor mod 20 = 0 and pycor mod 25 = 0  ]                                                               ;ask patches at every xy multiple of 25 to set their depth to -2
    [set DEPTH -2

    ]
    ask patches with [pxcor < 5] [set DEPTH 0]                                                                               ;set the source depth to 0

  ]


  [
    ;this block was used to generate a random landscape which was converted into a set csv file

    file-open "depth_for_rand_landscape.txt"
    while [not file-at-end?]
    [
      let next-X file-read
      let next-Y file-read
      let next-depth file-read
      ask patch next-x next-y [set depth next-depth]
    ]
    ask patches with [depth < 0] [set pcolor black]
    ask patches with [depth >= 0] [set pcolor blue + 2]
    ask patches with [pxcor < 5] [set pcolor blue - 2]

  ]

  file-close-all


end


;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;world ends;;;;;;;;;;;;;;world ends;;;;;;;;;;;;;;world ends;;;;;;;;;;;world ends;;;;;;;;;;;;;;;;;;world ends;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

to apocalypse_check                                                                                                          ;kill every fish not in a pool at the end of the dispersal window
  if ticks >= (DISPERSAL_WINDOW)
  [ ask turtles with [DEPTH >= 0]
    [die]
  ]

end

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;turtle creation;;;;;;;;;;;;;turtle creation;;;;;;;;;;;;;turtle creation;;;;;;;;;;turtle creation;;;;;;;;;;;;;;;;;turtle creation;;;;;;;;;;;;;;;;;;;;;;;;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;



to source
  setxy (random-float 5) (random max-pycor)
end


to make_fish

;  if EXPERIMENT_SETTINGS? = TRUE ;THIS BLOCK OF CODE IS CALLED DURING EXPERIMENTS. IT REMOVES ALL AESTHETIC PROPERTIES OF THE FISH TO OPTIMIZE MODEL RUN SPEED
  ;[
    set DIST_COUNT 0                                                             ;this code block is used to count the number of SOCIABILITY distributions selected.
    if social? = true [set DIST_COUNT DIST_COUNT + 1]
    if asocial? = true [set DIST_COUNT DIST_COUNT + 1]
    if hetero? = true [set DIST_COUNT DIST_COUNT + 1]


    if Control? = true [set DIST_COUNT DIST_COUNT + 1]







    if Social? = TRUE
      [create-socials (START_FISH_COUNT / DIST_COUNT)
        ask socials
        [    set shape "fish"
           set color blue
          set size 3
          set ALPHA 10
          set BETA 2
          source
          rngs:init
          let stream_id random-float 9999
          let seed random-float 9999
          rngs:set-seed stream_id seed
          let dist rngs:rnd-beta stream_id ALPHA BETA
          set SOCIABILITY dist

        ]
      ]

    if hetero? = TRUE
    [create-heterogeneouss (START_FISH_COUNT / DIST_COUNT)
      ask heterogeneouss
        [
           set shape "fish"
           set color scale-color green sociability 0 1
          set size 3
          set ALPHA 2
          set BETA 2
          source
          rngs:init
          let stream_id random-float 9999
          let seed random-float 9999
          rngs:set-seed stream_id seed
          let dist rngs:rnd-beta stream_id ALPHA BETA
          set SOCIABILITY dist

        ]
    ]

    if asocial? = TRUE
    [create-asocials (START_FISH_COUNT / DIST_COUNT)
      ask asocials
        [    set shape "fish"
           set color red
          set size 3
          set ALPHA 2
          set BETA 10
          source
          rngs:init
          let stream_id random-float 9999
          let seed random-float 9999
          rngs:set-seed stream_id seed
          let dist rngs:rnd-beta stream_id ALPHA BETA
          set SOCIABILITY dist

        ]
    ]


    if Control? = TRUE
    [create-Controls (START_FISH_COUNT / DIST_COUNT)
      ask Controls
        [    set shape "fish"
           set color black
          set size 3
          set SOCIABILITY 1.1
          source
        ]
    ]
 ; ]





  ask turtles [
    set TARGET_DENSITY (1 + round (SOCIABILITY * MAX_TARGET))                         ;creates the individual target DENSITY value.
    set MAX_SEARCH_RADIUS (MAX_MOVES_PER_TICK / 4)
    set heading 90]
  ranking
  if DIST_COUNT = 0
    [ crt 1]



end


to ranking
  let rank-list sort-on [SOCIABILITY] turtles                                                                 ;create a low to high sorted list of ranked SOCIABILITY scores
  let ranks n-values length rank-list [ ? ]                                                                   ;rename each turtle according to it's ranked location in that list
    (foreach rank-list ranks [ask ?1 [set SOCIAL_RANK ?2] ] )
end

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;plots & counters;;;;;;;;;;;;;plots & counters;;;;;;;;;;;;;plots & counters;;;;;;;;;;plots & counters;;;;;;;;;;;;;;plots & counters;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

to update_plots

  set-current-plot "SOCIABILITY distribution"                                              ;draw a histogram of the population sociabilities
  histogram [SOCIABILITY] of turtles


  set-current-plot "Individual target DENSITY"                                             ;draw a histogram of the target densities of the population, if any fish are alive
  if count turtles > 0[
    set-plot-x-range 0 (round max [TARGET_DENSITY] of turtles + 1)
    histogram [TARGET_DENSITY] of turtles
  ]

  if ticks > 0                                                                             ;after tick 0
  [
    set-current-plot "Pool DENSITY"                                                        ;draw a histogram of the pool densities
    if (count (patches with [DEPTH < 0 AND DENSITY > 0])) > 0
      [
        set-plot-x-range 0 (max ([DENSITY] of patches with [DEPTH < 0 AND DENSITY > 0] ) )
        if(count (patches with [DEPTH < 0 AND DENSITY > 0 ]) > 0)
        [histogram [DENSITY] of patches with [DEPTH < 0 AND DENSITY > 0]]

      ]

  ]

  if ticks >= DISPERSAL_WINDOW
  [let happy (count turtles with [MY_DENSITY > TARGET_DENSITY * (1 - DENSITY_WIGGLE_ROOM) AND MY_DENSITY < TARGET_DENSITY * (1 + DENSITY_WIGGLE_ROOM)])

    set CURRENT_COUNT count turtles

    ;this block produces the output placed in the white output box on the interface

    output-print (word (( CURRENT_COUNT / START# ) * 100) "% of turtles survived")
    output-print (word " ")
    IFelse happy > 0
      [ output-print (word " ")
        output-print (word ((happy / CURRENT_COUNT ) * 100) ("% of survivors achieved desired DENSITY"))  ]
      [ output-print (word " ")
        output-print (word 0 ("% of survivors achieved desired DENSITY"))  ]

    if CURRENT_COUNT > 0
    [ output-print (word ((mean [LINEAR_DISTANCE_TRAVELED] of turtles) * 100) "(m) mean linear distance traveled " )
      output-print (word " ")
      output-print (word ((mean [TOTAL_DISTANCE_TRAVELED] of turtles) * 100) "(m) mean total distance traveled")
      output-print (word " ")
      output-print (word (mean ([DENSITY] of patches with [DENSITY > 0 AND DEPTH < 0])) " mean DENSITY of occupied patches")
      output-print (word " ")
      output-print (word (standard-deviation ([DENSITY] of patches with [DENSITY > 0 and DEPTH < 0])) "standard deviation of the DENSITY of occupied patches")
      output-print (word " ")
      output-print (word ((mean ([TARGET_DENSITY] of turtles)) ) "mean TARGET_DENSITY ")
      output-print (word " ")
      output-print (word (standard-deviation ([TARGET_DENSITY] of turtles)) "standard deviation of the target DENSITY of fish")
      output-print (word " ")
      output-print (word (median ([DENSITY] of patches with [DENSITY > 0 and DEPTH < 0])) " median DENSITY of occupied patches")
      output-print (word " ")
      output-print (word ((MEDIAN ([TARGET_DENSITY] of turtles)) ) "MEDIAN TARGET_DENSITY ")
      output-print (word " ")
      output-print (word (modes ([DENSITY] of patches with [DEPTH < 0 AND DENSITY > 0])) " DENSITY modes of occupied patches")
      output-print (word " ")
      output-print (word ((MODES [TARGET_DENSITY] of turtles) ) "MODAL TARGET_DENSITY ")
    ]
  ]

end



;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;misc;;;;;;;;;;;;miscs;;;;;;;;;;;;;misc;;;;;;;;;;misc;;;;;;;;;misc;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
to set_globals
  if USE_OPTIMAL_SETTINGS? = TRUE                                                                     ;these values were selected using a sensativity analysis as they procude the desired patterns
  [set DENSITY_WIGGLE_ROOM 0.25
    set BIG_SEARCH_RADIUS 225
    set SMALL_SEARCH_RADIUS 133
    set MAX_HANG_OUT_TIME 5
    set SOCIAL_RANK_ORDERED_MOVEMENT? TRUE
    set WATCH_A_RANDOM_FISH? FALSE
    set START_FISH_COUNT 1000
    ; set DISPERSAL_WINDOW 200
    set MAX_TARGET 40
    set MAX_MOVES_PER_TICK 30
    set CUT_RUN_SHORT? FALSE
  ]

end

to Look_at_one_fish
  if watch_a_RANDOM_FISH? = TRUE
  [
    if watch_a_RANDOM_FISH?   = TRUE
    [
      ifelse user-yes-or-no? "Do you want to shrink all fish except the one of interest? They will continue to exist, but be invisible. This will allow the model to run faster, as well as make it easier to observe the fish of interest."
      [ask turtles [set size 0 ]

        set RANDOM_FISH random START_FISH_COUNT
        ask turtle RANDOM_FISH [set size 5 set color yellow pd]
        inspect turtle RANDOM_FISH
        output-print "Pull the inspection window down"
        output-print " "
        output-print "The white area around the fish you're watching is the"
        output-print " area searched in one day"
        output-print " "
        output-print "The command center shows this fish's local density"
        output-print "at the end of every day"
        output-print "............................................................... "
      ]
      [
        set RANDOM_FISH random START_FISH_COUNT
        ask turtle RANDOM_FISH [set size 5 set color yellow pd ]
        inspect turtle RANDOM_FISH
        output-print "Pull the inspection window down"
        output-print " "
        output-print "The white area around the fish you're watching is the"
        output-print " area searched in one day"
        output-print " "
        output-print "The command center shows this fish's local density"
        output-print "at the end of every day"
        output-print "................................................................. "

      ]
    ]
  ]

end

to fish_initial_state                                                                                                       ;this procedure is called during setup and sets all turtle parameters to the necessary initial state
  ask turtles
  [ set LAST_PATCH patch-here
    set FIRST_PATCH patch-here
    set TOTAL_DISTANCE_TRAVELED 0
    SET LINEAR_DISTANCE_TRAVELED 0
    Set TIME_SPENT_HERE 0
    Set STAY? FALSE
    set HANG_OUT_COUNTER 0
    set PANIC_TIME (DISPERSAL_WINDOW * 0.10)
  ]
end

to final_tick_procedures                                                                                                  ;this procedure is called at the end of the run to color fish based on their happieness and then start the apocalypse and update plots
  if Show_social_v_asocial_path? = TRUE
  [ask patches [set pcolor white]
    ask turtles with [size > 1]
      [
        ifelse sociability > 0.5
          [set label ""
            set plabel precision TOTAL_DISTANCE_TRAVELED 2
            ;ask one-of neighbors  [set plabel  "Social"]
            ask patch 60 60 [set plabel-color black set plabel "Social fish traveled an average (in m) of" ]
            ask patch 60 55 [set plabel-color black set plabel precision (100 * mean [TOTAL_DISTANCE_TRAVELED] of turtles with [sociability > 0.5]) 2]

          ]

          [set label ""
            set plabel precision TOTAL_DISTANCE_TRAVELED 2
            ;  ask one-of neighbors  [set plabel "Asocial"]
            ask patch 160 60 [set plabel-color black set plabel "Asocial fish traveled an average (in m) of" ]
            ask patch 160 55 [set plabel-color black set plabel precision (100 * mean [TOTAL_DISTANCE_TRAVELED] of turtles with [sociability < 0.5001]) 2]
          ]
      ]

  ]

  ask patches [set DENSITY count turtles-here]

  set DISTANCE_LIST (list "sociability distance")
  ask turtles [
    set DISTANCE_LIST lput "," DISTANCE_LIST
    set DISTANCE_LIST lput (precision SOCIABILITY 4) DISTANCE_LIST
    set DISTANCE_LIST lput " " DISTANCE_LIST
    set DISTANCE_LIST lput TOTAL_DISTANCE_TRAVELED DISTANCE_LIST
  ]

  ask one-of patches [set DENSITY_LIST (list "x density")]
  ask patches with [DEPTH < 0][
    set DENSITY_LIST lput " " DENSITY_LIST
    set DENSITY_LIST lput pxcor DENSITY_LIST
    set DENSITY_LIST lput " " DENSITY_LIST
    set DENSITY_LIST lput DENSITY DENSITY_LIST
  ]



  ask patches with [depth < 0 AND DENSITY > 0]  [set MEAN_SOCIABILITY (precision (mean [sociability] of turtles-here) 4)]
  ask patches with [DEPTH < 0 AND DENSITY = 0] [set MEAN_SOCIABILITY (0)]
  ask one-of patches [set SOCIABILITY_LIST (list "x y mean sociability")]
  ask patches with [DEPTH < 0][
    set SOCIABILITY_LIST lput "," SOCIABILITY_LIST
    set SOCIABILITY_LIST lput pxcor SOCIABILITY_LIST
    set SOCIABILITY_LIST lput " " SOCIABILITY_LIST
    set SOCIABILITY_LIST lput pYcor SOCIABILITY_LIST
    set SOCIABILITY_LIST lput " " SOCIABILITY_LIST
    set SOCIABILITY_LIST lput MEAN_SOCIABILITY SOCIABILITY_LIST
  ]



  ; ask patches with [DEPTH < 0] [set DENSITY_LIST (list (pxcor)(",") (DENSITY) (",") )]

;
;  ask patches with [DEPTH < 0 and DENSITY > 1] [set VARIANCE_LIST (list (pxcor)(",")(pycor) (",") (variance [SOCIABILITY] of turtles-here) (",") )]
;  ask patches with [DEPTH < 0 and DENSITY <= 1] [set VARIANCE_LIST (list (pxcor)(",")(pycor) (",") (0) (",") )]
;
;  ask patches with [DEPTH < 0 and DENSITY > 1] [set STDEV_LIST (list (pxcor)(",")(pycor) (",") (standard-deviation [SOCIABILITY] of turtles-here) (",") )]
;  ask patches with [DEPTH < 0 and DENSITY <= 1] [set STDEV_LIST (list (pxcor)(",")(pycor) (",") (0) (",") )]
;
  apocalypse_check
  set CURRENT_COUNT count turtles


end


to track_travel                                                                                                              ;this is invoked during go to tell fish to keep track of where they are with reference to where they just were as well as where they started
  if patch-here != LAST_PATCH
  [set TOTAL_DISTANCE_TRAVELED (TOTAL_DISTANCE_TRAVELED + (distance LAST_PATCH))
    set LAST_PATCH patch-here]
  set TOTAL_DISTANCE_TRAVELED precision TOTAL_DISTANCE_TRAVELED 4

  if ticks >= DISPERSAL_WINDOW [set LINEAR_DISTANCE_TRAVELED (distance FIRST_PATCH)]
  ; if Control? = TRUE AND ticks > 50 AND ((count turtles with [DEPTH < 0 ]) = (count turtles)) [set CUT_RUN_SHORT? TRUE]
end


to recolor_world
  ask patches with [depth >= 0] [set pcolor black]
  let MaxD (max [density] of patches with [depth < 0])
  ask patches with [depth < 0]
  [set plabel DENSITY
    ask other patches in-radius 5
    [set DENSITY (max [DENSITY] of patches in-radius 5 with [DEPTH < 0])
      set pcolor scale-color DENSITY red 0 MaxD
    ]
  ]
  ask patches with [depth < 0 AND density = 0] [ask other patches in-radius 5 [set pcolor white]]

  ask turtles [set size 0]

end

to recolor_world_by_sociability
  ask patches with [depth >= 0] [set pcolor black]
  ask patches with [DEPTH < 0 and DENSITY > 0] [set pcolor scale-color blue mean [SOCIABILITY] of turtles-here 0 1
    set plabel (precision (mean [sociability] of turtles-here) 2)
    let p pcolor
    ask patches in-radius 5 [set pcolor p] ]

  ask patches with [DEPTH < 0 AND DENSITY = 0]
  [
    set pcolor white
    ask other patches in-radius 5 [set pcolor white]
  ]

  ask turtles [set size 0]

end
@#$#@#$#@
GRAPHICS-WINDOW
212
136
880
497
-1
-1
3.274
1
10
1
1
1
0
0
0
1
0
200
0
100
0
0
1
day
30.0

BUTTON
5
10
69
43
NIL
Setup
NIL
1
T
OBSERVER
NIL
NIL
NIL
NIL
1

BUTTON
75
10
138
43
NIL
Go
T
1
T
OBSERVER
NIL
NIL
NIL
NIL
1

BUTTON
146
10
209
43
Step
Go
NIL
1
T
OBSERVER
NIL
NIL
NIL
NIL
1

SWITCH
-3
45
95
78
Social?
Social?
1
1
-1000

SWITCH
-3
78
95
111
Hetero?
Hetero?
1
1
-1000

SWITCH
-1
113
95
146
Asocial?
Asocial?
0
1
-1000

PLOT
208
10
368
130
Sociability distribution
NIL
NIL
0.0
1.0
0.0
1.0
true
false
"" ""
PENS
"" 0.01 1 -16777216 true "" ""

MONITOR
97
43
167
88
#_of_Fish
Count turtles
0
1
11

INPUTBOX
96
88
212
148
Dispersal_window
200
1
0
Number

INPUTBOX
96
150
211
210
Max_target
40
1
0
Number

PLOT
368
10
537
130
Individual target density
NIL
NIL
-0.1
10.0
0.0
1.0
true
false
"" ""
PENS
"default" 1.0 1 -16777216 true "" ""

INPUTBOX
96
211
211
271
Max_moves_per_tick
30
1
0
Number

SWITCH
591
527
799
560
Watch_a_random_fish?
Watch_a_random_fish?
1
1
-1000

INPUTBOX
96
272
210
332
Start_Fish_count
1000
1
0
Number

PLOT
536
10
696
130
Pool density
Density
# of pools
0.0
10.0
0.0
1.0
true
false
"" ""
PENS
"default" 1.0 1 -16777216 true "" ""

SLIDER
210
527
386
560
Density_wiggle_room
Density_wiggle_room
0
1
0.25
0.01
1
NIL
HORIZONTAL

SWITCH
591
495
799
528
Social_rank_ordered_movement?
Social_rank_ordered_movement?
0
1
-1000

MONITOR
699
11
803
56
Number of pools
Count patches with [depth < 0]
0
1
11

MONITOR
700
60
802
105
Mean D of pools
mean [count turtles-here] of patches with [depth < 0]
0
1
11

SLIDER
384
495
591
528
Big_search_radius
Big_search_radius
0
360
225
1
1
NIL
HORIZONTAL

SLIDER
384
528
591
561
Small_search_radius
Small_search_radius
0
360
133
1
1
NIL
HORIZONTAL

SWITCH
0
146
95
179
Control?
Control?
1
1
-1000

SLIDER
384
561
591
594
MAX_HANG_OUT_TIME
MAX_HANG_OUT_TIME
0
Dispersal_window
5
1
1
NIL
HORIZONTAL

SWITCH
384
593
591
626
Use_optimal_settings?
Use_optimal_settings?
0
1
-1000

SWITCH
-3
332
209
365
Grid_or_Rand_landscape?
Grid_or_Rand_landscape?
1
1
-1000

BUTTON
-2
366
184
399
Color pool region by density
Recolor_world
NIL
1
T
OBSERVER
NIL
NIL
NIL
NIL
1

BUTTON
-2
398
184
431
Color pool region by sociability
recolor_world_by_sociability
NIL
1
T
OBSERVER
NIL
NIL
NIL
NIL
1

TEXTBOX
1415
207
1565
263
There is an issue with the density_list function, use the densities of fish over space instead in mean time
11
0.0
1

SWITCH
-1
430
213
463
Show_social_v_asocial_path?
Show_social_v_asocial_path?
1
1
-1000

@#$#@#$#@
#Sociability dependent dispersal ODD
#Model created by: Jesse R. Blanchard
#Blanchard.Jesse@Gmail.com


#Overview
##Purpose
The primary purpose of this model is to demonstrate one mechanism by which intraspecific variability of individual personalities, in this case sociability, can influence the distribution and abundance of a single species, via colonization mediation, during re-assembly. The secondary purpose is to develop personality dependent dispersal modeling theory upon which more complex models can be built.
This model addresses the question: Does a spatially structured population emerge from individual variation in sociability?
####Specifically:
Does a population lacking sociability dependent dispersal occupy a broader range (more patches) than populations making socially dependent dispersal decisions?
Do populations with heterogeneous sociability distributions disperse further than homogenous populations?
Which sociability distribution offers the highest rate of survival in an ephemeral landscape with short, moderate, or long dispersal windows?


##Entities, state variables & scales
###Entities:
This model is designed to simulated an ephemeral floodplain type system where pools are distributed along a landscape which is seasonally connected to a single source population. This model will have two entities: fish (turtles) and patches.
####State variables:
####Fish:
Each fish will have one state variable of sociability, which places them in to one of six categories based on the range of their sociability scores: extreme social, social, asocial, extreme asocial, heterogeneous or null. All personality scores will be drawn randomly from a normal distribution and be bound to greater than 0.5 for social, less than 0.5 for asocial and from 0 – 1 for heterogeneous. This sociability score will determine individual preferred conspecific density which each individual will seek to achieve by

Preferred density=sociability score*maximum  target density

where the acceptable local density range is set by

Preferred density*0.75  < local density < Preferred density*1.25

They are able to move a maximum of 3,000m in a single day (tick), which has been estimated to be a reasonable maximum linear dispersal distance for Gambusia holbrooki, a small-bodied poeciliid fish typically used in sociability studies (Cote et al. 2011), in the Everglades floodplain’s (Gatto et al. UPD). Fish are also characterized by their discrete location and a “hang out” probability which reduces by 10% each day the fish is in a pool that does not fall within the acceptable local density range. When queried, each fish will be able to report their total linear distance, and full total distance traveled during the run.
####Patches:
Patches will have one categorical state variable of depth, which in this simplified model will be either 0 or -2. When depth is less than 0 a patch is considered a viable settlement patch, henceforth called pools. When it is not then fish are able to cross the patch but will not consider staying there. At the end of the run, any fish outside a patch with depth less than 0 will die.
####Spatial scale:
This model uses a generalized form of the Rocky Glades region of the Everglades ecosystem as a model ephemeral floodplain with a single source (Shark River Slough). All fish will start with an x coordinate less than 5 and a random y coordinate, to represent starting in the source area. The world itself is bound as a box, and consists of 200 * 100 regular square patches each representing 100m of the model system. Therefore, when scaled to reality this model simulates a 20,000 * 10,000m marsh with the source at the western edge of the system. The real marsh is approximately 20km wide. While the Rocky Glades themselves are taller than 10km, it is reasonable to assume that few fish exhibit the rheotaxis necessary to move too distantly North, while the southern edge can be used to represent the Main Park Road which likely serves as a dispersal barrier for most fishes.


##Process overview and scheduling
When the model is initialized, pools are arranged in a grid of size 25 starting at the corner-origin patch and all pools are assigned a depth of -1, except for those patches within the source range. This results in a total of 40 pools interspersed evenly throughout an unsuitable dispersal matrix.  No environmental gradients are imposed in this version of the model. While this detracts from the reality of the model system, additional complexity would distract from the purpose of this first model. 1000 fish will be generated with one of six sociability beta-distributions: social, extremely social, asocial, extremely asocial, heterogeneous or null (Figure 1), in the source area using the parameters found in Table 1. In the extreme asocial population all fish will have a sociability of 0.000001 to approximate 0, while avoiding mathematical errors later in the model. Conversely, extreme social fish will have a sociability of 1 and null fish will lack any social information. The extreme and null scenarios will lack intra-specific heterogeneity.
####Table 1 Parameters used to generate random beta distributions from which sociability scores are drawn for the population.
      Population	α	  Β
      Social	    10	   2
      Heterogeneous	2	  2
      Asocial	    2	   10


On each tick, day, fish will move in the bottom-up ranked order of their sociability scores, in accordance with the conceptual model put forth by Cote et al. (2010). Movement will be comprised of two sequential processes: pool searching and conspecific evaluation, as well as a final overriding survival protocol. If a pool is determined to be unsatisfactory, then a sub-procedure of the conspecific evaluation will determine (with 10% daily reductions in probability) if the fish will remain or leave.

###Pool searching
Every fish is required to find a pool before it can make any other decisions. A fish does this by first checking if the patch it is on, or those within their maximum search radius (0.25 * the maximum daily linear distance they can move), has a depth less than 0. If it is then they center themselves on that pool and evaluate the conspecific density (except null fish, which simply stop at the pool forever). If they’re not starting the day in a pool, then they observe the surrounding area and if there is a pool nearby they go there and evaluate the conspecific density. If no pools are nearby, then they begin a circular search procedure. If they have found a pool recently, but it lacked an acceptable density, the fish will use a tight search procedure. However, if too long passes without finding a new pool the fish will use a broader search pattern. When they find a pool, they will evaluate the conspecific density. A key feature of this search protocol is that the fish remembers the last pool it evaluated and will not settle in to it again until it finds a different pool, unless the survival protocol is invoked. This prevents the fish from getting ‘stuck’ continually returning to and re-evaluating a single hole.
###Survival
As the ultimate goal of fish searching for pools in an ephemeral wetland is presumably to survive the dry down, and it is known that they will seek aquatic refugia (e.g. pools, sloughs, etc.) at the end of the wet season in the Rocky Glades (Kobza et al. 2004), then all other procedures will be overridden by survival near the end of the model run. In this case, they will use the same pool searching procedure, except when they find a pool they will stay there regardless of conspecific density to avoid being caught on the marsh surface at the end of the run (in the dry season). Any fish not in a pool at the end of the run die.
###Should I stay?
A fish decides if it should stay in a pool by first evaluating the conspecific density, considering any value between ±0.25 * Preferred density as acceptable. If the density is acceptable, then the fish will stay where it is and set the probability of remaining to maximum. If the pool is unacceptable then the fish will decide daily, with a daily 10% reduction in probability, if they should stay to wait for conditions to improve. The initial value of the staying probability is calculated as the maximum allowable wait time (1 month) * their individual sociability scores such that asocial fishes have a higher probability of leaving sooner than social fishes. If a fish decides to abandon the pool it will note the location of the pool, and on the following day it will begin the pool search procedure anew. If it re-encounters that same pool before finding a new one then the pool will be ignored; however, the fish’s memory will only extend to the most recently evaluated pool.

#Design concepts
###	Basic principles
Individual personalities have been demonstrated to influence the dispersal probability and distance of populations, thereby influencing their distributions in space and time (Duckworth and Badyaev 2007; Tristan et al. 2014). In a metacommunity, spatial heterogeneity in species distributions is often explained in part as competition/colonization trade-offs and niche differences between species which sort along environmental or community gradients (Mouquet and Loreau 2003; Holyoak et al. 2005; Kadowaki et al. 2011; Altermatt 2012; Sokol et al. 2014). However, as articulated by Cote et al. (2010a), if individual personalities can influence the spatial distribution of a metapopulation, and if individuals of all species have some form of personality, then it stands to reason that individual personalities can influence the spatial distribution of species in a metacommunity. The implications of which being that these personalities then have a direct impact on the diversity, distribution and abundance of species within a metacommunity (Cote et al. 2010a) regardless of prevailing metacommunity paradigm. As these are frequently studied emergent properties of metacommunities, the inclusion of behavioral data will then logically enhance our understanding of these factors as well as spatial ecology at large. However, as of yet the authors are unaware of any clear demonstration of the effect of individual personalities on key metapopulation, or metacommunity, parameters in the absence of additional confounding information. The purpose of this model is to serve as that theoretical basis upon which future, more complex, works can build.

The behavioral type used in this model, sociability, is one which has frequently been demonstrated to have significant impacts on animal dispersal (Cote et al. 2010b; Cote et al. 2010a; Sih et al. 2012; Myles-Gonzalez et al. 2015), which is at the core of metacommunity theory (Holyoak et al. 2005). In this context, sociability is the propensity of an individual to associate with conspecifics, such that in low conspecific densities they disperse more frequently than in high densities (Conrad et al. 2011). While behavioral types frequently occur in complex behavioral syndromes (i.e. asocial animals tend to also be aggressive and bold), sociability alone has been repeatedly shown to influence an individual’s likelihood of dispersing (Conrad et al. 2011) and therefore is more relevant to the purposes of this model than the rest of the syndrome.
In the Rocky Glades study system’s late dry season fishes are concentrated in high density communities within Shark River Slough, and very few other smaller sources (Loftus et al. 1992; Goss et al. 2014). When the wet season begins the marsh floods and fishes disperse from the source, which marks the western boundary, in to the marsh. While the reasons have yet to be definitively shown, fishes invade the marsh surface. During this time, they are thought to forage and seek more optimal conditions. When the dry season begins these fish are then forced to assemble in to aquatic refugia, solution holes (henceforth referred to as pools for simplicity), thereby forming new communities. This presents the opportunity to seek preferable social situations by preferentially settling in to pools with the optimal conspecific density. As all fish start dispersing from the same source at the same time, it is likely that asocial individuals will be forced to disperse further in order to optimize their local conspecific density. While all dispersal models away from a single source in the absence of environmental gradients are likely to produce a negative relationship between density and distance, this scenario would result in a less negative relationship between asocial density and distance when compared to social density and distance. Populations lacking this social information would then serve as a null model test of these influences, and likely produce an inverse relationship more negative than asocial populations but less so than socials. While these behavioral types as well as their influences on population distributions and abundances have been well documented in nature, there has yet to be a demonstration that these impacts persist in the absence of any other potential, possibly cryptic, drivers and thus the implications for metapopulation assembly are still poorly understood. This model seeks to describe the properties of abundance and distribution which emerge from individual variations of behavioral types during metapopulation assembly.
For the purposes of this model, sociability and boldness will both be scored on a continuous scale of 0-1. This scale is derived from the way in which the trait is assessed experimentally, by dividing the time spent associating with conspecifics (or exploring in the case of boldness) by the total trial time. Each species in the model has a beta distribution of each behavioral type using the parameters found in Table 1. While it has also been shown that inter and intraspecific interactions can significantly influence all of the target characters (Holyoak et al. 2005; Society 2011; Kadowaki et al. 2011; Meldrum 2012; Heino et al. 2015), this model strives to demonstrate the sole effect of personality and therefore ignores additional influences.


###Emergence
This model strives to demonstrate how patterns of metapopulation distribution and abundance can emerge from individual variation in personality. Specifically, it will show variable relationships between the densities of populations with different sociability distributions and distance.
###Adaptation
At each time step, each individual must decide whether or not to move based on the depth of their current patch, and if in a pool then the decision is based on their own sociability as it relates to local and regional conspecific densities over time.
Objectives
Their survival objective is to seek aquatic refugia, pools, before the end of the wet season (model run). This is manifested in two ways. First fish disperse from the source in search of pools spread throughout an unsuitable dispersal habitat. When a pool is found, the sociability objective is then optimized. Second, if a fish is still dispersing by the final 10% of the wet season it will abandon any evaluations of the pools themselves and settle in to the first pool it finds so as to avoid death (which is the fate of any fish not in a pool at the end of the run).
Their sociability objective is to optimize local conspecific density by abandoning found pools, during the first 90% of the run time, which do not contain an acceptable conspecific density. However, to account for the possibility of an unacceptable pool becoming acceptable, the fish will wait for a short time before abandoning the pool to search for a new one.
###Learning
The only learning fish are capable of is the ability to remember the last pool they evaluated by refusing to spend time evaluating/settling in to that pool, and to keeping track of how long it has been since they settled in to an unacceptable pool as well as how long they spend dispersing after abandoning a pool. They forget the pool after another is found. No prior knowledge of the system is inherent at the start of the run.

###Sensing
Each individual fish has total knowledge of their maximum search radius, which is 25% of the maximum linear distance it can move. This is assumed to be the area a fish can intensively search in a single day; however, future works should evaluate this assumption.
###Interaction
The only interaction between fish is mediated through pool density.
Stochasticity
When no specific direction is determined by objective seeking, movement direction and distance is determined using random number generators under various constraints. Individual sociability is also drawn randomly from a beta distribution, except for the extreme populations which have fixed values and null populations which lack personalities.
###Collectives
The formation of population collectives will emerge from individual fish’s decisions regarding settlement, as part of their conspecific density optimization objective, and survival.
###Observation
####To evaluate if the model is operating properly, the model will output:
    The percentage of fish which survived.
    The percentage of survivors which successfully attained their target conspecific density.
    Mean, median, modes and standard deviation of the density of occupied pools.
    Mean, median, modes and standard deviation of the target density of fish.
    Mean total and linear distance traveled by all surviving fish.

Additionally, for better visualization of a single fish’s actions, an option is provided which shrinks all fish except one random fish which is then inspected, watched, asked to draw its path and highlight its total search radius each day.

####To demonstrate that individual personalities can increase the spatial structuring of a metapopulation, the model will output:
	Density mode and mean of occupied pools in each column of pools (ex. X=25, x=50, etc.)
	Mean linear distance traveled by individuals
	Mean total distance traveled by individuals

These values will be compared between all six sociability distributions with the expectation that null fish, will have higher densities nearer the source with lower mean and total distance’s traveled as well as a much more negative relationship between distance and mean density.

#Details
###Initialization
The model is initialized with pools arranged in a grid of size 25 starting at the corner origin, with the exception of patches within 5 of the western boundary. This western area is the source, within which 1000 fish are randomly distributed. These fish are randomly assigned sociability scores from a beta distribution with the parameters found in Table 1. If multiple distributions are requested simultaneously, an equal proportion of the population will receive each sociability distribution. The maximum daily traversable distance is set to 30 patches, which represents 3km and has been found in recent studies to be a reasonable maximum potential daily movement limit for Gambusia holbrooki (Gatto dissertation, UPD), the most common fish in the model system (Blanchard dissertation UPD, Kobza et al. 2004).
###Input data
This model does not rely on any external data.
#####	Sub-models
####Null_pool_search
If a null population is called, this procedure tells the null population to first make note of where they are with reference to where they have been in the past. If a fish starts the day in a pool, it centers itself in the pool and stops. If a fish starts outside a pool it first evaluates if the local depth is greater than any of the other patches in the searchable radius, and if any of those other patches are pools. If they are, then the fish moves to one of the patches in the searchable radius with the minimum depth of the area and stops. If the local patch and surrounding patches are not pools, then it engages in the new_pool_search procedure.
####New_pool_search
If the time since the last new pool has been found is less than the predetermined panic time, 10% of the total run time, then the fish will turn to the right at a random real angle less than 180o. If not, then the fish will turn right at a random real angle less than 360 o. After turning the fish will move at a random real distance less than the maximum distance a single fish can move in a single day.
####Pool_search
For all non-null distributions, this procedure called during Go tells the population to first make note of where they are with reference to where they have been in the recent past, and if they’re in a pool to decide if they should stay using the should_I_hang_out procedure. If the fish is not in a pool it will look to see if there is one nearby, if there is it will go there. If not, then it will run new_pool_search.
####Should_I_hang_out
This procedure decides if a fish is ‘happy’. In this context, happy is defined as having a local density within 25% of the individual’s target density (sociability * maximum target density). If the fish is happy, it will set stay where it is. If not, then it will randomly decide if it should stay or not. The maximum probability of staying on a patch is determined by the global MAX_HANG_OUT_TIME * SOCIABILITY, with a daily 10% decrease. This counter is compared to a randomly drawn real number. If the randomly drawn number is greater than the current value of the counter, the fish will abandon the patch on the following day.
####Survive
When less than 10% of the model run is left all fish will abandon all other procedures in favor of this procedure. Fish ask if they’re in a pool. If they are, they stop and will stop permanently. If they’re not in a pool, but one of the patches in their search radius is a pool then they’ll go there and stop moving permanently. If they haven’t found a pool, then they’ll use the new_pool_search procedure to find one. This procedure ignores conspecific density.
##References
#### .
    Altermatt F (2012) Temperature-related shifts in butterfly phenology depend on the habitat. Glob Chang Biol 18:2429–2438. doi: 10.1111/j.1365-2486.2012.02727.x
    Conrad JL, Weinersmith KL, Brodin T, et al (2011) Behavioural syndromes in fishes: a review with implications for ecology and fisheries management. J Fish Biol 78:395–435. doi: 10.1111/j.1095-8649.2010.02874.x
    Cote J, Clobert J, Brodin T, et al (2010a) Personality-dependent dispersal: characterization, ontogeny and consequences for spatially structured populations. Philos Trans R Soc Lond B Biol Sci 365:4065–76. doi: 10.1098/rstb.2010.0176
    Cote J, Fogarty S, Brodin T, et al (2011) Personality-dependent dispersal in the invasive mosquitofish: group composition matters. Proc R Soc Biol Sci 278:1670–8. doi: 10.1098/rspb.2010.1892
    Cote J, Fogarty S, Weinersmith K, et al (2010b) Personality traits and dispersal tendency in the invasive mosquitofish (Gambusia affinis). Proc R Soc Biol Sci 277:1571–9. doi: 10.1098/rspb.2009.2128
    Duckworth R a., Badyaev A V (2007) Coupling of dispersal and aggression facilitates the rapid range expansion of a passerine bird. Proc Natl Acad Sci U S A 104:15017–22. doi: 10.1073/pnas.0706174104
    Goss CW, Loftus WF, Trexler JC (2014) Seasonal Fish Dispersal in Ephemeral Wetlands of the Florida Everglades. Wetlands January:1–11. doi: 10.1007/s13157-013-0375-3
    Heino J, Nokela T, Soininen J, et al (2015) Elements of metacommunity structure and community- environment relationships in stream organisms. Freshw Biol 60:973–988. doi: 10.1111/fwb.12556
    Holyoak M, Liebold MA, Holt RD (2005) Metacommunities.
    Kadowaki K, Leschen R a. B, Beggs JR (2011) Competition-colonization dynamics of spore-feeding beetles on the long-lived bracket fungi Ganoderma in New Zealand native forest. Oikos 120:776–786. doi: 10.1111/j.1600-0706.2011.19302.x
    Kobza RM, Trexler JC, Loftus WF, Perry SA (2004) Community structure of fishes inhabiting aquatic refuges in a threatened Karst wetland and its implications for ecosystem management. Biol Conserv 116:153–165. doi: 10.1016/S0006-3207(03)00186-1
    Loftus WF, Johnson RA, Anderson GH (1992) Ecological impacts of the reduction of groundwater levels in short-hydroperiod marshes of the Everglades. In: Proceedings of the First International Converence on Ground Water Ecology. pp 199–208
    Meldrum GE (2012) Interactive effects of dispersal rate and disturbance synchrony on microarthropod diversity at multiple spatial scales: resuce effects do not increase regional richness. The University of British Colombia
    Mouquet N, Loreau M (2003) Community patterns in source-sink metacommunities. Am Nat 162:544–57. doi: 10.1086/378857
    Myles-Gonzalez E, Burness G, Yavno S, et al (2015) To boldly go where no goby has gone before: boldness, dispersal tendency, and metabolism at the invasion front. Behav Ecol. doi: 10.1093/beheco/arv050
    Sih A, Cote J, Evans M, et al (2012) Ecological implications of behavioural syndromes. Ecol Lett 15:278–89. doi: 10.1111/j.1461-0248.2011.01731.x
    Society E (2011) Competition-Colonization Trade-offs and Disturbance Effects at Multiple Scales  88:823–829.
    Sokol ER, Hoch JM, Gaiser E, Trexler JC (2014) Metacommunity Structure Along Resource and Disturbance Gradients in Everglades Wetlands. Wetlands 34:135–146. doi: 10.1007/s13157-013-0413-1
    Tristan J, Cucherousset J, Cote J (2014) Animal personality and the ecological impacts of freshwater non-native species across levels of biological organizations.
@#$#@#$#@
default
true
0
Polygon -7500403 true true 150 5 40 250 150 205 260 250

airplane
true
0
Polygon -7500403 true true 150 0 135 15 120 60 120 105 15 165 15 195 120 180 135 240 105 270 120 285 150 270 180 285 210 270 165 240 180 180 285 195 285 165 180 105 180 60 165 15

arrow
true
0
Polygon -7500403 true true 150 0 0 150 105 150 105 293 195 293 195 150 300 150

box
false
0
Polygon -7500403 true true 150 285 285 225 285 75 150 135
Polygon -7500403 true true 150 135 15 75 150 15 285 75
Polygon -7500403 true true 15 75 15 225 150 285 150 135
Line -16777216 false 150 285 150 135
Line -16777216 false 150 135 15 75
Line -16777216 false 150 135 285 75

bug
true
0
Circle -7500403 true true 96 182 108
Circle -7500403 true true 110 127 80
Circle -7500403 true true 110 75 80
Line -7500403 true 150 100 80 30
Line -7500403 true 150 100 220 30

butterfly
true
0
Polygon -7500403 true true 150 165 209 199 225 225 225 255 195 270 165 255 150 240
Polygon -7500403 true true 150 165 89 198 75 225 75 255 105 270 135 255 150 240
Polygon -7500403 true true 139 148 100 105 55 90 25 90 10 105 10 135 25 180 40 195 85 194 139 163
Polygon -7500403 true true 162 150 200 105 245 90 275 90 290 105 290 135 275 180 260 195 215 195 162 165
Polygon -16777216 true false 150 255 135 225 120 150 135 120 150 105 165 120 180 150 165 225
Circle -16777216 true false 135 90 30
Line -16777216 false 150 105 195 60
Line -16777216 false 150 105 105 60

car
false
0
Polygon -7500403 true true 300 180 279 164 261 144 240 135 226 132 213 106 203 84 185 63 159 50 135 50 75 60 0 150 0 165 0 225 300 225 300 180
Circle -16777216 true false 180 180 90
Circle -16777216 true false 30 180 90
Polygon -16777216 true false 162 80 132 78 134 135 209 135 194 105 189 96 180 89
Circle -7500403 true true 47 195 58
Circle -7500403 true true 195 195 58

circle
false
0
Circle -7500403 true true 0 0 300

circle 2
false
0
Circle -7500403 true true 0 0 300
Circle -16777216 true false 30 30 240

cow
false
0
Polygon -7500403 true true 200 193 197 249 179 249 177 196 166 187 140 189 93 191 78 179 72 211 49 209 48 181 37 149 25 120 25 89 45 72 103 84 179 75 198 76 252 64 272 81 293 103 285 121 255 121 242 118 224 167
Polygon -7500403 true true 73 210 86 251 62 249 48 208
Polygon -7500403 true true 25 114 16 195 9 204 23 213 25 200 39 123

cylinder
false
0
Circle -7500403 true true 0 0 300

dot
false
0
Circle -7500403 true true 90 90 120

face happy
false
0
Circle -7500403 true true 8 8 285
Circle -16777216 true false 60 75 60
Circle -16777216 true false 180 75 60
Polygon -16777216 true false 150 255 90 239 62 213 47 191 67 179 90 203 109 218 150 225 192 218 210 203 227 181 251 194 236 217 212 240

face neutral
false
0
Circle -7500403 true true 8 7 285
Circle -16777216 true false 60 75 60
Circle -16777216 true false 180 75 60
Rectangle -16777216 true false 60 195 240 225

face sad
false
0
Circle -7500403 true true 8 8 285
Circle -16777216 true false 60 75 60
Circle -16777216 true false 180 75 60
Polygon -16777216 true false 150 168 90 184 62 210 47 232 67 244 90 220 109 205 150 198 192 205 210 220 227 242 251 229 236 206 212 183

fish
false
0
Polygon -1 true false 44 131 21 87 15 86 0 120 15 150 0 180 13 214 20 212 45 166
Polygon -1 true false 135 195 119 235 95 218 76 210 46 204 60 165
Polygon -1 true false 75 45 83 77 71 103 86 114 166 78 135 60
Polygon -7500403 true true 30 136 151 77 226 81 280 119 292 146 292 160 287 170 270 195 195 210 151 212 30 166
Circle -16777216 true false 215 106 30

flag
false
0
Rectangle -7500403 true true 60 15 75 300
Polygon -7500403 true true 90 150 270 90 90 30
Line -7500403 true 75 135 90 135
Line -7500403 true 75 45 90 45

flower
false
0
Polygon -10899396 true false 135 120 165 165 180 210 180 240 150 300 165 300 195 240 195 195 165 135
Circle -7500403 true true 85 132 38
Circle -7500403 true true 130 147 38
Circle -7500403 true true 192 85 38
Circle -7500403 true true 85 40 38
Circle -7500403 true true 177 40 38
Circle -7500403 true true 177 132 38
Circle -7500403 true true 70 85 38
Circle -7500403 true true 130 25 38
Circle -7500403 true true 96 51 108
Circle -16777216 true false 113 68 74
Polygon -10899396 true false 189 233 219 188 249 173 279 188 234 218
Polygon -10899396 true false 180 255 150 210 105 210 75 240 135 240

house
false
0
Rectangle -7500403 true true 45 120 255 285
Rectangle -16777216 true false 120 210 180 285
Polygon -7500403 true true 15 120 150 15 285 120
Line -16777216 false 30 120 270 120

leaf
false
0
Polygon -7500403 true true 150 210 135 195 120 210 60 210 30 195 60 180 60 165 15 135 30 120 15 105 40 104 45 90 60 90 90 105 105 120 120 120 105 60 120 60 135 30 150 15 165 30 180 60 195 60 180 120 195 120 210 105 240 90 255 90 263 104 285 105 270 120 285 135 240 165 240 180 270 195 240 210 180 210 165 195
Polygon -7500403 true true 135 195 135 240 120 255 105 255 105 285 135 285 165 240 165 195

line
true
0
Line -7500403 true 150 0 150 300

line half
true
0
Line -7500403 true 150 0 150 150

pentagon
false
0
Polygon -7500403 true true 150 15 15 120 60 285 240 285 285 120

person
false
0
Circle -7500403 true true 110 5 80
Polygon -7500403 true true 105 90 120 195 90 285 105 300 135 300 150 225 165 300 195 300 210 285 180 195 195 90
Rectangle -7500403 true true 127 79 172 94
Polygon -7500403 true true 195 90 240 150 225 180 165 105
Polygon -7500403 true true 105 90 60 150 75 180 135 105

plant
false
0
Rectangle -7500403 true true 135 90 165 300
Polygon -7500403 true true 135 255 90 210 45 195 75 255 135 285
Polygon -7500403 true true 165 255 210 210 255 195 225 255 165 285
Polygon -7500403 true true 135 180 90 135 45 120 75 180 135 210
Polygon -7500403 true true 165 180 165 210 225 180 255 120 210 135
Polygon -7500403 true true 135 105 90 60 45 45 75 105 135 135
Polygon -7500403 true true 165 105 165 135 225 105 255 45 210 60
Polygon -7500403 true true 135 90 120 45 150 15 180 45 165 90

sheep
false
15
Circle -1 true true 203 65 88
Circle -1 true true 70 65 162
Circle -1 true true 150 105 120
Polygon -7500403 true false 218 120 240 165 255 165 278 120
Circle -7500403 true false 214 72 67
Rectangle -1 true true 164 223 179 298
Polygon -1 true true 45 285 30 285 30 240 15 195 45 210
Circle -1 true true 3 83 150
Rectangle -1 true true 65 221 80 296
Polygon -1 true true 195 285 210 285 210 240 240 210 195 210
Polygon -7500403 true false 276 85 285 105 302 99 294 83
Polygon -7500403 true false 219 85 210 105 193 99 201 83

square
false
0
Rectangle -7500403 true true 30 30 270 270

square 2
false
0
Rectangle -7500403 true true 30 30 270 270
Rectangle -16777216 true false 60 60 240 240

star
false
0
Polygon -7500403 true true 151 1 185 108 298 108 207 175 242 282 151 216 59 282 94 175 3 108 116 108

target
false
0
Circle -7500403 true true 0 0 300
Circle -16777216 true false 30 30 240
Circle -7500403 true true 60 60 180
Circle -16777216 true false 90 90 120
Circle -7500403 true true 120 120 60

tree
false
0
Circle -7500403 true true 118 3 94
Rectangle -6459832 true false 120 195 180 300
Circle -7500403 true true 65 21 108
Circle -7500403 true true 116 41 127
Circle -7500403 true true 45 90 120
Circle -7500403 true true 104 74 152

triangle
false
0
Polygon -7500403 true true 150 30 15 255 285 255

triangle 2
false
0
Polygon -7500403 true true 150 30 15 255 285 255
Polygon -16777216 true false 151 99 225 223 75 224

truck
false
0
Rectangle -7500403 true true 4 45 195 187
Polygon -7500403 true true 296 193 296 150 259 134 244 104 208 104 207 194
Rectangle -1 true false 195 60 195 105
Polygon -16777216 true false 238 112 252 141 219 141 218 112
Circle -16777216 true false 234 174 42
Rectangle -7500403 true true 181 185 214 194
Circle -16777216 true false 144 174 42
Circle -16777216 true false 24 174 42
Circle -7500403 false true 24 174 42
Circle -7500403 false true 144 174 42
Circle -7500403 false true 234 174 42

turtle
true
0
Polygon -10899396 true false 215 204 240 233 246 254 228 266 215 252 193 210
Polygon -10899396 true false 195 90 225 75 245 75 260 89 269 108 261 124 240 105 225 105 210 105
Polygon -10899396 true false 105 90 75 75 55 75 40 89 31 108 39 124 60 105 75 105 90 105
Polygon -10899396 true false 132 85 134 64 107 51 108 17 150 2 192 18 192 52 169 65 172 87
Polygon -10899396 true false 85 204 60 233 54 254 72 266 85 252 107 210
Polygon -7500403 true true 119 75 179 75 209 101 224 135 220 225 175 261 128 261 81 224 74 135 88 99

wheel
false
0
Circle -7500403 true true 3 3 294
Circle -16777216 true false 30 30 240
Line -7500403 true 150 285 150 15
Line -7500403 true 15 150 285 150
Circle -7500403 true true 120 120 60
Line -7500403 true 216 40 79 269
Line -7500403 true 40 84 269 221
Line -7500403 true 40 216 269 79
Line -7500403 true 84 40 221 269

wolf
false
0
Polygon -16777216 true false 253 133 245 131 245 133
Polygon -7500403 true true 2 194 13 197 30 191 38 193 38 205 20 226 20 257 27 265 38 266 40 260 31 253 31 230 60 206 68 198 75 209 66 228 65 243 82 261 84 268 100 267 103 261 77 239 79 231 100 207 98 196 119 201 143 202 160 195 166 210 172 213 173 238 167 251 160 248 154 265 169 264 178 247 186 240 198 260 200 271 217 271 219 262 207 258 195 230 192 198 210 184 227 164 242 144 259 145 284 151 277 141 293 140 299 134 297 127 273 119 270 105
Polygon -7500403 true true -1 195 14 180 36 166 40 153 53 140 82 131 134 133 159 126 188 115 227 108 236 102 238 98 268 86 269 92 281 87 269 103 269 113

x
false
0
Polygon -7500403 true true 270 75 225 30 30 225 75 270
Polygon -7500403 true true 30 75 75 30 270 225 225 270

@#$#@#$#@
NetLogo 5.3.1
@#$#@#$#@
@#$#@#$#@
@#$#@#$#@
<experiments>
  <experiment name="SETTINGS_OPTIMIZATION_EXP" repetitions="5" runMetricsEveryStep="false">
    <setup>setup</setup>
    <go>go</go>
    <metric>[density] of patches with [depth &lt; 0]</metric>
    <metric>standard-deviation [density] of patches with [depth &lt; 0]</metric>
    <metric>modes [density] of patches with [depth &lt; 0]</metric>
    <metric>median [density] of patches with [depth &lt; 0]</metric>
    <metric>mean [linear_distance_traveled] of turtles</metric>
    <metric>mean [total_distance_traveled] of turtles</metric>
    <enumeratedValueSet variable="Asocial?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="E_Asocial?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Social?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <steppedValueSet variable="Big_search_radius" first="150" step="50" last="350"/>
    <enumeratedValueSet variable="Small_search_radius">
      <value value="100"/>
      <value value="150"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Max_target">
      <value value="40"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="E_Social?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Dispersal_window">
      <value value="200"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Social_rank_ordered_movement?">
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Null?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Max_moves_per_tick">
      <value value="30"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Hetero?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <steppedValueSet variable="Density_wiggle_room" first="0" step="0.25" last="0.5"/>
    <steppedValueSet variable="MAX_HANG_OUT_TIME" first="20" step="20" last="60"/>
    <enumeratedValueSet variable="Watch_a_random_fish?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Start_Fish_count">
      <value value="1000"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="EXPERIMENT_SETTINGS?">
      <value value="true"/>
    </enumeratedValueSet>
  </experiment>
  <experiment name="performance_evaluation" repetitions="1000" runMetricsEveryStep="false">
    <setup>setup</setup>
    <go>go</go>
    <metric>[density] of patches with [depth &lt; 0]</metric>
    <metric>standard-deviation [density] of patches with [depth &lt; 0]</metric>
    <metric>modes [density] of patches with [depth &lt; 0]</metric>
    <metric>median [density] of patches with [depth &lt; 0]</metric>
    <metric>mean [linear_distance_traveled] of turtles</metric>
    <metric>mean [total_distance_traveled] of turtles</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor = 25]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor = 50]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor = 75]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor = 100]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor = 125]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor = 150]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor = 175]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor = 200]</metric>
    <enumeratedValueSet variable="Use_optimal_settings?">
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Experiment_settings?">
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="E_Social?">
      <value value="true"/>
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Null?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="E_Asocial?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Hetero?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Social?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Asocial?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
  </experiment>
  <experiment name="patch color" repetitions="1" runMetricsEveryStep="false">
    <setup>setup</setup>
    <go>go</go>
    <metric>[depth] of patches with [pycor &lt;= 20]</metric>
    <metric>[depth] of patches with [pycor &lt;= 40 AND pycor &gt; 20]</metric>
    <metric>[depth] of patches with [pycor &lt;= 60 AND pycor &gt; 40]</metric>
    <metric>[depth] of patches with [pycor &lt;= 80 AND pycor &gt; 60]</metric>
    <metric>[depth] of patches with [pycor &lt;= 100 AND pycor &gt; 80]</metric>
    <enumeratedValueSet variable="E_Social?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Grid_or_Rand_landscape?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Max_moves_per_tick">
      <value value="30"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Watch_a_random_fish?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="MAX_HANG_OUT_TIME">
      <value value="33"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Experiment_settings?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Social_rank_ordered_movement?">
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Control?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Social?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Null?">
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Start_Fish_count">
      <value value="1000"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Dispersal_window">
      <value value="2"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Use_optimal_settings?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Density_wiggle_room">
      <value value="0.25"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Small_search_radius">
      <value value="133"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Max_target">
      <value value="40"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Hetero?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="E_Asocial?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Asocial?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Big_search_radius">
      <value value="225"/>
    </enumeratedValueSet>
  </experiment>
  <experiment name="sensativity analysis for hang out time" repetitions="5" runMetricsEveryStep="false">
    <setup>setup</setup>
    <go>go</go>
    <metric>mean [linear_distance_traveled] of turtles</metric>
    <metric>mean [total_distance_traveled] of turtles</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor &lt;= 25]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor &lt;= 50 AND Pxcor &gt;= 25]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor &lt;= 75 AND Pxcor &gt;= 50]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor &lt;= 100 AND Pxcor &gt;= 75]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor &lt;= 125 AND Pxcor &gt;= 75]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor &lt;= 150 AND Pxcor &gt;= 125]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor &lt;= 175 AND Pxcor &gt;= 150]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor &gt; 176]</metric>
    <enumeratedValueSet variable="Start_Fish_count">
      <value value="1000"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Watch_a_random_fish?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Null?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Big_search_radius">
      <value value="225"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="E_Social?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Grid_or_Rand_landscape?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Dispersal_window">
      <value value="200"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Density_wiggle_room">
      <value value="0.25"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Hetero?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Max_target">
      <value value="40"/>
    </enumeratedValueSet>
    <steppedValueSet variable="MAX_HANG_OUT_TIME" first="0" step="5" last="35"/>
    <enumeratedValueSet variable="Small_search_radius">
      <value value="133"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Social?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="E_Asocial?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Control?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Max_moves_per_tick">
      <value value="30"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Experiment_settings?">
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Use_optimal_settings?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Asocial?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Social_rank_ordered_movement?">
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Bimodal?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
  </experiment>
  <experiment name="density experiment grid" repetitions="1000" runMetricsEveryStep="false">
    <setup>setup</setup>
    <go>go</go>
    <metric>mean [linear_distance_traveled] of turtles</metric>
    <metric>mean [total_distance_traveled] of turtles</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor &lt;= 25]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor &lt;= 50 AND Pxcor &gt; 25]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor &lt;= 75 AND Pxcor &gt;= 50]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor &lt;= 100 AND Pxcor &gt;= 75]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor &lt;= 125 AND Pxcor &gt;= 100]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor &lt;= 150 AND Pxcor &gt;= 125]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor &lt;= 175 AND Pxcor &gt;= 150]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor &gt; 175]</metric>
    <enumeratedValueSet variable="Start_Fish_count">
      <value value="1000"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Watch_a_random_fish?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Big_search_radius">
      <value value="225"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Dispersal_window">
      <value value="200"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Density_wiggle_room">
      <value value="0.25"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Max_target">
      <value value="40"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="MAX_HANG_OUT_TIME">
      <value value="5"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Small_search_radius">
      <value value="133"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Max_moves_per_tick">
      <value value="30"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Experiment_settings?">
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Use_optimal_settings?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Social_rank_ordered_movement?">
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Grid_or_Rand_landscape?">
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="E_Asocial?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Asocial?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Bimodal?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Hetero?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Social?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="E_Social?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Null?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Control?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
  </experiment>
  <experiment name="density experiment rand" repetitions="1000" runMetricsEveryStep="false">
    <setup>setup</setup>
    <go>go</go>
    <metric>mean [linear_distance_traveled] of turtles</metric>
    <metric>mean [total_distance_traveled] of turtles</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor &lt;= 25]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor &lt;= 50 AND Pxcor &gt; 25]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor &lt;= 75 AND Pxcor &gt;= 50]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor &lt;= 100 AND Pxcor &gt;= 75]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor &lt;= 125 AND Pxcor &gt;= 100]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor &lt;= 150 AND Pxcor &gt;= 125]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor &lt;= 175 AND Pxcor &gt;= 150]</metric>
    <metric>[density] of patches with [depth &lt; 0 AND Pxcor &gt; 175]</metric>
    <enumeratedValueSet variable="Start_Fish_count">
      <value value="1000"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Watch_a_random_fish?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Big_search_radius">
      <value value="225"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Dispersal_window">
      <value value="200"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Density_wiggle_room">
      <value value="0.25"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Max_target">
      <value value="40"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="MAX_HANG_OUT_TIME">
      <value value="5"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Small_search_radius">
      <value value="133"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Max_moves_per_tick">
      <value value="30"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Experiment_settings?">
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Use_optimal_settings?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Social_rank_ordered_movement?">
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Grid_or_Rand_landscape?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="E_Asocial?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Asocial?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Bimodal?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Hetero?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Social?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="E_Social?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Null?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Control?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
  </experiment>
  <experiment name="sociability experiment" repetitions="1000" runMetricsEveryStep="false">
    <setup>setup</setup>
    <go>go</go>
    <metric>[SOCIABILITY_LIST] of patches with [depth &lt; 0]</metric>
    <metric>[DENSITY_LIST] of patches with [depth &lt; 0]</metric>
    <enumeratedValueSet variable="E_Social?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Asocial?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Social_rank_ordered_movement?">
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Hetero?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Social?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Control?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Start_Fish_count">
      <value value="1000"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Use_optimal_settings?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Big_search_radius">
      <value value="225"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Grid_or_Rand_landscape?">
      <value value="true"/>
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Watch_a_random_fish?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Max_target">
      <value value="40"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Density_wiggle_room">
      <value value="0.25"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Null?">
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Dispersal_window">
      <value value="60"/>
      <value value="120"/>
      <value value="200"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Max_moves_per_tick">
      <value value="30"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Experiment_settings?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Bimodal?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="E_Asocial?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="MAX_HANG_OUT_TIME">
      <value value="5"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Small_search_radius">
      <value value="133"/>
    </enumeratedValueSet>
  </experiment>
  <experiment name="sociability variance sdev density" repetitions="1000" runMetricsEveryStep="false">
    <setup>setup</setup>
    <go>go</go>
    <metric>[DENSITY_LIST] of patches with [DEPTH &lt; 0]</metric>
    <metric>[SOCIABILITY_LIST] of patches with [DEPTH &lt; 0]</metric>
    <metric>[STDEV_LIST] of patches with [DEPTH &lt; 0]</metric>
    <metric>[VARIANCE_LIST] of patches with [DEPTH &lt; 0]</metric>
    <enumeratedValueSet variable="Watch_a_random_fish?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Null?">
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="E_Social?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="E_Asocial?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Asocial?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Small_search_radius">
      <value value="133"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Grid_or_Rand_landscape?">
      <value value="true"/>
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="MAX_HANG_OUT_TIME">
      <value value="5"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Start_Fish_count">
      <value value="1000"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Control?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Use_optimal_settings?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Max_target">
      <value value="40"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Max_moves_per_tick">
      <value value="30"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Social_rank_ordered_movement?">
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Dispersal_window">
      <value value="60"/>
      <value value="120"/>
      <value value="200"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Density_wiggle_room">
      <value value="0.25"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Bimodal?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Experiment_settings?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Social?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Big_search_radius">
      <value value="225"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Hetero?">
      <value value="false"/>
    </enumeratedValueSet>
  </experiment>
  <experiment name="fixed sociability list output" repetitions="1000" runMetricsEveryStep="false">
    <setup>setup</setup>
    <go>go</go>
    <metric>[DENSITY_LIST] of patches with [DEPTH &lt; 0]</metric>
    <metric>[SOCIABILITY_LIST] of patches with [DEPTH &lt; 0]</metric>
    <metric>[STDEV_LIST] of patches with [DEPTH &lt; 0]</metric>
    <enumeratedValueSet variable="Dispersal_window">
      <value value="60"/>
      <value value="120"/>
      <value value="200"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Hetero?">
      <value value="true"/>
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Null?">
      <value value="true"/>
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Asocial?">
      <value value="true"/>
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Social?">
      <value value="true"/>
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Control?">
      <value value="true"/>
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Bimodal?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="E_Social?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="E_Asocial?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="MAX_HANG_OUT_TIME">
      <value value="5"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Grid_or_Rand_landscape?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Watch_a_random_fish?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Small_search_radius">
      <value value="133"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Big_search_radius">
      <value value="225"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Start_Fish_count">
      <value value="1000"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Use_optimal_settings?">
      <value value="false"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Max_target">
      <value value="40"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Max_moves_per_tick">
      <value value="30"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Social_rank_ordered_movement?">
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Density_wiggle_room">
      <value value="0.25"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Experiment_settings?">
      <value value="true"/>
    </enumeratedValueSet>
  </experiment>
  <experiment name="POST MAP EXPERIMENT" repetitions="1000" runMetricsEveryStep="false">
    <setup>setup</setup>
    <go>go</go>
    <metric>DENSITY_LIST</metric>
    <metric>SOCIABILITY_LIST</metric>
    <metric>DISTANCE_LIST</metric>
    <enumeratedValueSet variable="Hetero?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Social?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Asocial?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Control?">
      <value value="false"/>
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Use_optimal_settings?">
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Experiment_settings?">
      <value value="true"/>
    </enumeratedValueSet>
    <enumeratedValueSet variable="Dispersal_window">
      <value value="20"/>
      <value value="200"/>
    </enumeratedValueSet>
  </experiment>
</experiments>
@#$#@#$#@
@#$#@#$#@
default
0.0
-0.2 0 0.0 1.0
0.0 1 1.0 0.0
0.2 0 0.0 1.0
link direction
true
0
Line -7500403 true 150 150 90 180
Line -7500403 true 150 150 210 180

@#$#@#$#@
0
@#$#@#$#@

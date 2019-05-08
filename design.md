# Overwatch Race Framework

## Global Variable Definitions


    A - powerup positions
    B - powerup spawn times
    C - powerup types
    
    D - Player drop positions
    E - Player drop despawn times
    F - Player drop types
    
    G - type to player state table 
    
    H
    I
    J
    K
    L 
    M
    N
    O
    P
    Q - Debug
    R - Enable/disable debug hud
    S
    T
    U
    V
    W
    X
    Y - Initialization completed
    Z - Global state machine state

## Player Variable Definitions

    A-G - Gun calculation intermediate values
    H - Orb damage intermediate
    I
    J
    K
    L
    M
    N - damage orb index
    O - damage orb 1
    P - damage orb 2
    Q - damage orb 3
    R - damage orb explode time array
    S
    T
    U
    V
    W - Player weapon
    X - requested player weapon
    Y - orbs finished spawning
    Z - Player state
    
## Global state machine

**global state 0 - initialization**

**global state 1 - set powerup positions**

**global state 2 - set powerup initial spawn times**

**global state 3 - set powerup types**

**global state 4 - global state 4 - create powerup type to state mapping**

## Player rules

**Weapon 4 - orb (copy 1) - on fire

    Cond: N == 0 //adjust per copy
    
    Settings:
        {max distance}: 46.2
        {initial spawn distance}: 1.3
        {speed}: 10
        {time between shots}: 1.5
        constraint: {time between shots} * 3 + {small buffer time} < (({max distance} - {initial spawn distance}) / {speed})
        constraint: {time between shots} is a multiple of 0.25 (to make things simple)
        
    A <= ray cast hit position( //hit position
        start pos: eye position(ep)
        end pos: eye position(ep) + facing direction(ep) * {max distance}
        players to include: none
        include player owned objects: false
    )
    B <= eye position(ep) + facing direction(ep) * {initial spawn distance} //initial spawn position
    C <= ( (distance between(B, A) / {speed} ) + 2 //time until despawn
    
    skip if( line of sight( eye position(ep), B ) ) { //don't shoot through walls
        wait 0.25 abort
        loop if true
        abort
    }
    F <= true //fire lock
    play effect ( explostion sound )
    play effect ( good explosion )
    
    //change O for P in copy 2 and Q in copy 3
    O <= B
    Chase player variable at rate(
        variable: O
        destination: A
        rate: {speed}
        reevaluation: none
    )
    
    //replace R[0] with R[1] in copy 2 and R[3] in copy 3
    R[0] <= C
    skip if ( not( {time between shots} < R[0] ) ) {
        //unlock first, then despawn
        wait {time between shots}
        play effect(debuff impact sound ep only)
        N <= 1 //adjust for copies
        F <= false
        skip if ( R[0] - {time between shots} < 0.25, 1)
        wait ( R[0] - {time between shots} ) restart if true
        stop chasing( O )
        O <= (-999,-999,-999)
        loop if true
        abort
    }
    // despawn first, then unlock
    wait R[0]
    stop chasing( O )
    O <= (-999,-999,-999)
    wait ( {time between shots} - R[0] )
    play effect(debuff impact sound ep only)
    N <= 1 //adjust for copies
    F <= false
    loop if true

**orb damage copy 1**

replace O for P in copy 2 and Q in copy 3

    cond: is true for any ( all players, cu:O != (-999,-999,-999) )
    
    settings:
        {damage area}
        {damage per 0.25s tic}
        
    H <= filtered array( //attackers with orb hitting player
        array: remove from array(all players, ep)
        condition: distance between( ep, cu:O ) < {damage area} ) 
    )
    
    skip if ( H == [] ) { //or empty array? or 0? Not sure
        damage(
            player: ep
            damager: random value in array( H )
            amount: {damage per 0.25s tic} * count of ( H )
        )
    }
    
    wait 0.25
    loop if true
            
    
    





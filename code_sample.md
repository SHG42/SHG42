```

//In states/gameState_pointer.js
update() {
    //...
    // run ledge finding function
    this.game.physics.arcade.overlap(this.hero, this.mapObjects.ledgesGroup, this.ledgeHit, null, this);
    //...
}

ledgeHit(hero, ledge) {
    if(!hero.custom.isGrabbing && hero.body.velocity.y < 0) {
        this.hero.custom.isGrabbing = true;
        this.hero.animations.currentAnim.stop(false, true);
        this.goUpBy = ledge.body.y - 13;
        this.getEndpoint(ledge);

        if (ledge.side === "left" && hero.custom.whichDirection === "left") {
            this.hero.animations.play('grab-left');
            hero.custom.grabLeft = true;
            hero.alignIn(ledge, Phaser.TOP_LEFT, 15, 5); //offset accounts for sprite bounding box
            hero.body.position.setTo(ledge.body.x, ledge.body.y);
            this.freeze();
        }
        if (ledge.side === "right" && hero.custom.whichDirection === "right") {
            this.hero.animations.play('grab-right');
            hero.custom.grabRight = true;
            hero.alignIn(ledge, Phaser.TOP_RIGHT, 15, 5); //offset accounts for sprite bounding box
            hero.body.position.setTo(ledge.body.x, ledge.body.y);
            this.freeze();
        }
    }
}

getEndpoint(ledge) {
    this.mapObjects.endsGroup.forEach((end)=>{
        if(end.id === ledge.end) {
            this.end = end;
            return this.end;
        }
    });
}

climbingLeft() { 
    this.addTweenUp();
    this.hero.input.enabled = false;
    this.hero.animations.play('climb-left');
}

climbingRight() {
    this.addTweenUp();
    this.hero.input.enabled = false;
    this.hero.animations.play('climb-right');
}

addTweenUp() {
    this.tweenUp = this.game.add.tween(this.hero).to({ y: this.goUpBy }, 1000, 'Circ.easeOut');
    this.tweenUp.onStart.add(()=>{ this.hero.input.enabled = false; }, this);
    this.tweenUp.onComplete.add(()=>{
        if(this.hero.custom.whichDirection === "left") {
            this.hero.animations.play('roll-left');
        } else if(this.hero.custom.whichDirection === "right") {
            this.hero.animations.play('roll-right'); 
        }
    }, this);
}

freeze() {
    this.hero.custom.isJumping = false;
    this.hero.custom.tapped = false;
    this.hero.input.enabled = true;
    this.hero.body.stop(); //set speed/accel/velo to 0
    this.hero.body.immovable = true; //no impact from other bodies
    this.hero.body.moves = false; //physics system does not move body, but can be moved manually 
    this.hero.body.enable = false; //won't be checked for any form of collision or overlap or have its pre/post updates run.
    this.hero.body.allowGravity = false; //body's local gravity disabled
    this.gravity = 815;
}

unfreeze0() {
    this.hero.body.immovable = false;
    this.hero.body.enable = true;
    this.hero.body.moves = true;
    this.hero.body.allowGravity = true;
}

unfreeze() {
    this.hero.body.reset();
    this.hero.body.stop();
    this.hero.custom.tapped = false;
    this.hero.custom.isGrabbing = false;
    this.hero.custom.isClimbing = false;
    this.hero.input.enabled = true;
    if(this.hero.custom.whichDirection === "left") {
        this.hero.animations.play('idle-left');
    } else if(this.hero.custom.whichDirection === "right") {
        this.hero.animations.play('idle-right'); 
    }
}

//In states/configurators/addHero.js
//grab
this.grabLeft = this.hero.animations.add('grab-left', Phaser.Animation.generateFrameNames('crnr-grb-left-', 0, 3, '-1.3', 2), 3, true, false);
this.grabRight = this.hero.animations.add('grab-right', Phaser.Animation.generateFrameNames('crnr-grb-right-', 0, 3, '-1.3', 2), 3, true, false);

//climb
this.climbLeft = this.hero.animations.add('climb-left', Phaser.Animation.generateFrameNames('crnr-clmb-left-', 0, 4, '-1.3', 2), 10, false, false);
this.climbRight = this.hero.animations.add('climb-right', Phaser.Animation.generateFrameNames('crnr-clmb-right-', 0, 4, '-1.3', 2), 10, false, false);

//roll
this.rollLeft = this.hero.animations.add('roll-left', Phaser.Animation.generateFrameNames('smrslt-left-', 0, 3, '-1.3', 2), 10, true, false);
this.rollRight = this.hero.animations.add('roll-right', Phaser.Animation.generateFrameNames('smrslt-right-', 0, 3, '-1.3', 2), 10, true, false);

//up from roll
this.slideLeft = this.hero.animations.add('slide-left', Phaser.Animation.generateFrameNames('slide-left-', 0, 4, '-1.3', 2), 10, false, false);
this.slideRight = this.hero.animations.add('slide-right', Phaser.Animation.generateFrameNames('slide-right-', 0, 4, '-1.3', 2), 10, false, false);

////LEFT-HAND ROLL
this.rollLeft.onStart.add(()=>{
    this.end = game.state.callbackContext.end;
    this.gravity = game.state.callbackContext.gravity;
    game.state.callbackContext.unfreeze0();
    this.hero.body.gravity.x = -this.gravity;
}, this);

this.rollLeft.onLoop.add(()=>{
    game.physics.arcade.moveToObject(this.hero, this.end);
    if(game.physics.arcade.collide(this.hero, this.end)) {
        this.rollLeft.stop(false, true);
    } else if(this.hero.body.blocked.left || this.hero.body.touching.left || this.hero.body.position.x <= this.end.body.position.x) {
        this.rollLeft.stop(false, true);
    }
}, this);

this.rollLeft.onComplete.add(()=> { this.hero.animations.play('slide-left'); }, this);

this.slideLeft.onComplete.add(()=>{ this.hero.body.speed = 0; game.state.callbackContext.unfreeze(); this.hero.body.gravity.x = 0; }, this);

this.climbLeft.onStart.add(()=>{ 
    game.state.callbackContext.tweenUp.start();
}, this);

////RIGHT-HAND ROLL
this.rollRight.onStart.add(()=>{
    this.end = game.state.callbackContext.end;
    this.gravity = game.state.callbackContext.gravity;
    game.state.callbackContext.unfreeze0();
    this.hero.body.gravity.x = this.gravity;
}, this);

this.rollRight.onLoop.add(()=>{
    game.physics.arcade.moveToObject(this.hero, this.end);
    if(game.physics.arcade.collide(this.hero, this.end)) {
        this.rollRight.stop(false, true);
    } else if(this.hero.body.blocked.right || this.hero.body.touching.right || this.hero.body.position.x >= this.end.body.position.x) {
        this.rollRight.stop(false, true);
    }
}, this);

this.rollRight.onComplete.add(()=> { this.hero.animations.play('slide-right'); }, this);

this.slideRight.onComplete.add(()=>{ this.hero.body.speed = 0; game.state.callbackContext.unfreeze(); this.hero.body.gravity.x = 0; }, this);

this.climbRight.onStart.add(()=>{
    game.state.callbackContext.tweenUp.start();
}, this);

```

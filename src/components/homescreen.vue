<template>
    <div class="page-container">
        <transition name="slide-left" mode="in-out">
            <div class="flex-container"
             v-bind:style="{ backgroundImage: 'url(/images/bg/' + backgroundNumber + '.png)' }">
                <div class="wrapper">
                    <div class="beta">
                        Alpha Build
                    </div>
                    <div class="content">
                            <div class="title" v-if="generatedGoal === false">
                                <h1>Runescape Goals</h1>
                                <p>Everything is progress</p>
                            </div>
                            <div class="tools" v-if="generatedGoal === false" >
                                <p>
                                I would like a
                                <span class="button working length" v-bind:class="{ active: this.goalLength === 'short' }"
                                v-on:click="toggleLength('short')">
                                Short</span>
                                /
                                <span class="button working length" v-bind:class="{ active: this.goalLength === 'long' }"
                                v-on:click="toggleLength('long')">
                                Long</span> term goal.
                                </p>
                                <br>
                                <p>I would like my goal to focus on:
                                    <ul>
                                        <li>
                                            <span class="button"
                                                    v-bind:class="{ active: this.goalFocus === 'quests' }"
                                            >
                                                <img src="/images/icon1.png" alt="quest icon"> Quests
                                            </span>
                                        </li>
                                        <li>
                                            <span class="button working" v-on:click="toggleFocus('skills')"
                                                    v-bind:class="{ active: this.goalFocus === 'skills' }"
                                            >
                                                <img src="/images/icon2.png" alt="Skills Icon"> Skills
                                            </span>
                                        </li>
                                        <li>
                                            <span class="button working" v-on:click="toggleFocus('minigames')"
                                                    v-bind:class="{ active: this.goalFocus === 'minigames' }"
                                            >
                                                <img src="/images/icon3.png" alt="Minigame Icon"> Minigames
                                            </span>
                                        </li>
                                        <li>
                                            <span class="button"
                                                v-bind:class="{ active: this.goalFocus === 'combat' }"
                                            >
                                                <img src="/images/icon4.png" alt="Combat icon"> Combat
                                            </span>
                                        </li>
                                        <li>
                                            <span class="button"
                                                    v-bind:class="{ active: this.goalFocus === 'strange' }"
                                            >
                                                <img src="/images/icon5.png" alt="Something Strange icon"> Something Strange...
                                            </span>
                                        </li>
                                    </ul>
                                </p>
                                <div type="button" name="button"
                                    v-on:click="generateGoal()"
                                    v-bind:class="{ active: this.isValid }"
                                    class="generateGoalButton">
                                    Generate My Goal!
                                </div>
                            </div>
                        <transition name="fade">
                            <div class="goal_display" v-if="generatedGoal === true">
                                <h2>{{this.goalTitle}}</h2>
                                <h3>{{this.goalSubTitle}}</h3>
                                <p>{{this.goalDesc}}</p>
                                <div class="completed" v-on:click="clearAll(true)">
                                    <i class="fas fa-check"></i>
                                    I have completed this!
                                    <i class="fas fa-check"></i>
                                </div>
                                <p class="back" v-on:click="clearAll()">
                                    I don't want this goal, let me try again!
                                </p>
                            </div>
                        </transition>
                    </div>
                </div>
            </div>
        </transition>
        <transition name="fade">
            <div class="completed-screen" v-if="goalCompleted">
                <h1>CONGRATULATIONS</h1>
                <div class="completed-dance"></div>
                <p>Well done! Now go do something productive!</p>
                <div class="close" v-on:click="goalCompleted = false">
                    <i class="fas fa-times"></i>
                    Close
                    <i class="fas fa-times"></i>
                </div>
            </div>
        </transition>
        <footer>
            <p> This is an Open Source Project - <a href="https://github.com/AllanOcelot/Runescape-Goals">Contribute Today!</a> &nbsp; </p>
            <p> Made by <a href="http://allancodes.com" target="_blank">allanCodes</a>. </p>
        </footer>
        <a class="donation_link" href="http://www.paypal.me/McKernan" target="_blank"><i class="fas fa-heart"></i> Please consider a donation <i class="fas fa-heart"></i> </a>

        <runescapeTwitchChannels />
    </div>
</template>

<script>
import axios from 'axios';
import runescapeTwitchChannels from './runescapeTwitchChannels.vue';

export default {
        name: 'homescreen',
        props: {
            msg: String
        },
        components: {
            runescapeTwitchChannels
        },
        data: function() {
            return {
                backgroundNumber: 1,
                goalLength: '',
                goalFocus: '',
                isValid: false,
                goalData: {},
                generatedGoal: false,
                goalTitle: '',
                goalSubTitle: '',
                goalDesc: '',
                goalCompleted: false,
            }
        },
        methods: {
            generateBackground: function() {
                this.backgroundNumber = Math.floor(Math.random() * Math.floor(6)) + 1;
            },
            toggleLength: function(length) {
                if (length === this.goalLength ){
                    this.goalLength = '';
                    this.checkIfValid();
                    return;
                }
                this.goalLength = length;
                this.checkIfValid();
            },
            toggleFocus: function(focus) {
                if (focus === this.goalFocus ){
                    this.goalFocus = '';
                    this.checkIfValid();
                    return;
                }
                this.goalFocus = focus;
                this.checkIfValid();
            },
            checkIfValid: function(){
                if(this.goalLength != '' && this.goalFocus != ''){
                    this.isValid = true;
                } else {
                    this.isValid = false;
                }
            },

            // Get the file
            async getJSON(filename){
                let json = await axios.get('/data/' +  filename + '.json');
                return json;
            },

            clearAll(completed){
                if(completed === true){
                    this.goalFocus = '';
                    this.goalLength = '';
                    this.isValid = false;
                    this.goalCompleted = true;
                }
                this.goalFocus = '';
                this.goalLength = '';
                this.goalDesc = '';
                this.generatedGoal = false;
            },


            generateGoal: function(){
                let self = this;
                if(this.isValid === true) {
                    //Call the filename into Data
                    this.getJSON(this.goalFocus).then( function(result) {
                        self.goalData = result.data;

                        //Generate a random number to pick what subset to use
                        var max_number =  self.goalData.data.length;
                        var sub_seed   =  Math.floor(Math.random() * Math.floor(max_number));

                        // I.E Skills->Attack ( This is the 'subset')
                        var subset_name     =   Object.keys(self.goalData.data[sub_seed])[0];
                        var subset_data     =   self.goalData.data[sub_seed][subset_name].tasks;


                        // Randomly Pick a task from the list
                        var taskID          =   Math.floor(Math.random() * Math.floor(subset_data.length));
                        var task            =   subset_data[taskID];
                        var task_subtitle   =   self.goalData.data[sub_seed][subset_name].title;
                        var task_desc       =   task.desc;

                        //Check if our task contains an 'X' action.
                        if( task_desc.includes("#") ){

                            //Generate a number depending if the user has picked long or short
                            var length_var = Math.floor(Math.random() * Math.floor(1000));
                            if (self.goalLength === 'short'){
                                length_var = Math.floor(Math.random() * Math.floor(100));
                            }
                            length_var = "" + length_var + "";
                            task_desc = task_desc.replace("#", length_var);
                        }

                        //Return the task
                        self.goalTitle     = subset_name;
                        self.goalSubTitle     = task_subtitle;
                        self.goalDesc      = task_desc;
                        self.generatedGoal = true;

                    });
                }
            }
        },
        created: function () {
            this.generateBackground();
        },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">

    .beta {
        position: fixed;
        top: 0px;
        left: 0px;
        width: 120px;
        height: 60px;
        line-height: 60px;
        background: red;
        z-index: 10;
        color: #fff;
        font-family: 'Catamaran', sans-serif;
        font-weight: 600;
        border-radius: 0 0 30px 0;
        opacity: 0.7;
    }

    .flex-container {
        float: left;
        position: relative;
        display: flex;
        align-items: center;
        justify-content: center;
        height: 100vh;
        min-height: 600px;
        width: 100%;
        background: #e1e1e1;
        color: #111;
        -moz-box-shadow:    inset 0 0 10px #000000;
        -webkit-box-shadow: inset 0 0 10px #000000;
        box-shadow:         inset 0 0 10px #000000;
        background-position: center;
        background-size: cover;
        background-repeat: no-repeat;
        -webkit-user-select: none; /* Chrome/Safari */
        -moz-user-select: none; /* Firefox */
        -ms-user-select: none; /* IE10+ */

        /* Rules below not implemented in browsers yet */
        -o-user-select: none;
        user-select: none;
    }

    .flex-container::after {
        content: "";
        position: absolute;
        left: 0;
        top: 0;
        width: 100%;
        height: 100%;
        background: #000;
        opacity: 0.85;
        z-index: 1;
    }

    .content {
        position: relative;
        color: #fff;
        z-index: 2;
    }


    /* Title */
    .title {
        position: relative;
        margin-bottom: 40px;
        h1 {
            font-family: 'Catamaran', sans-serif;
            padding: 0;
            margin: 0;
            font-size: 2.4em;
        }
        p {
            position: relative;
            padding: 0;
            margin: 0 auto;
            width: 160px;
        }
        p::before,
        p::after {
            content: "";
            position: absolute;
            left: -65px;
            top: 8px;
            width: 55px;
            height: 1px;
            background: #fff;
            opacity: 0.8;
            z-index: 1;
        }
        p::after {
        left: auto;
        right: -65px;
        }
    }
    


    /* The goal tools */
    .flex-container .tools {
        p {
            margin: 0 0 10px 0;
            cursor: default;
        }

        ul {
            display: block;
            width: 100%;
            text-align: center;
            margin: 15px 0 20px 0;
            padding: 0;

            li {
                list-style: none;
                display: inline-block;
            }
        }

        span.button {
            position: relative;
            z-index: 10;
            padding: 2px 12px;
            margin: 0 5px;
            cursor: pointer;
            background: none;
            border: 1px solid #fff;
            color: #fff;
            font-family: 'Catamaran', sans-serif;
            font-weight: 400;
            line-height: normal;
            border-radius: 4px;
            transition: all 0.3s;

            /* Short term, remove once all buttons function */
            opacity: 0.3;

            /* Buttons with class working, work */
            &.working {
                opacity: 1;
            }

            /* Make the length buttons fully visible, as they work  */
            &.length {
                opacity: 1;
            }

            img {
                position: relative;
                top: 2px;
            }

            &:hover {
                cursor: not-allowed;
            }

            &.active,
            &.active:hover,
            &.working:hover {
                cursor: pointer;
                background: #32af41;
                color: #fff;
                border-radius: 4px;
                opacity: 1;
            }
        }

        @media screen and (max-width: 850px){
            span.button {
                padding: 0px 6px;
            }
        }
    }
    



    /* Generate Goal Button -- Disabled until user fills out questions */
    .generateGoalButton {
        width: 280px;
        height: 44px;
        line-height: 46px;
        letter-spacing: 1px;
        border-radius: 0;
        border: none;
        background: transparent;
        border: 1px solid #fff;
        border-radius: 2px;
        color: #fff;
        font-size: 1.12em;
        font-weight: 600;
        opacity: 0.5;
        font-family: 'Catamaran', sans-serif;
        margin: 30px auto 0 auto;
        cursor: not-allowed;
        background-image: linear-gradient(#ffffff00,#ffffff00);
        transition: all 0.2s;

        &:hover {
            cursor: not-allowed;
            opacity: 0.6;
        }
        &.active {
            cursor: pointer;
            opacity: 1;
            &:hover {
                opacity: 1;
                background: #32af41;
                color: #fff;
            }
        }
    }



    /* The Goal */
    .goal_display {
        position: fixed;
        left: 50%;
        top: 50%;
        margin-left: -200px;
        margin-top: -200px;
        padding: 20px;
        background-image: url('/images/goal_bg.png');
        background-size: contain;
        background-repeat: no-repeat;
        background-position: center;
        color: #faff00;
        text-shadow: 2px 2px 0px rgba(0,0,0,0.6);
        height: auto;
        box-sizing: border-box;
        z-index: 20;
        display: flex;
        justify-content: center;
        align-items: center;
        flex-direction: column;
        width: 400px;
        height: 400px;

        h2 {
            display: block;
            width: 100%;
            font-size: 1.6em;
            margin: 0 0 10px 0;
            padding: 0;
            font-family: 'Catamaran', sans-serif;
            cursor: default;
        }

        h3 {
            margin-top: 0;
            margin-bottom: 10px;
            cursor:default;
            text-transform: capitalize;
        }

        p {
            margin: 0;
            line-height: 24px;
            font-size: 14px;
            padding: 0 20px;
            cursor: default;
        }

        .completed {
            height: 40px;
            padding: 0 10px;
            box-sizing: border-box;
            line-height: 40px;
            letter-spacing: 1px;
            border-radius: 0;
            border: none;
            background: transparent;
            border: 1px solid #fff;
            text-shadow: none;
            border-radius: 4px;
            color: #fff;
            font-size: 1em;
            font-weight: 400;
            margin-top: 15px;
            font-family: 'Catamaran', sans-serif;
            margin: 30px auto 10px auto;
            transition: all 0.3s;

            &:hover {
                background: #32af41;
                color: #fff;
                cursor: pointer;
            }
        }

        .back {
            position: relative;
            bottom: -8px;
            left: 0px;
            text-align: center;
            width: 100%;
            width: 300px;
            font-size: 13px;
            text-shadow: none;
            color: #fff;
            cursor: pointer;
            &:hover {
                text-decoration: underline;
            }
        }

    }

    /* Completed screen */
    .completed-screen {
        position: fixed;
        top: 0;
        left: 0;
        width: 100vw;
        height: 100vh;
        background: #040204;
        z-index: 100;
        padding: 20px;
        box-sizing: border-box;
        display: flex;
        flex-direction: column;
        align-content: center;
        justify-content: center;

        h1 {
            font-size: 3em;
            letter-spacing: 2px;
        }
        
        p {
            font-size: 1.3em;
        }

        .close {
            width: 200px;
            margin: 0 auto;
            line-height: 54px;
            height: 50px;
            font-size: 1.2em;
            text-transform: uppercase;
            letter-spacing: 2px;
            border-radius: 6px;
            border: 2px solid #fff;
            transition: all 0.2s;
            i {
                position: relative;
                margin: 0 5px;
                top: 4px;
                font-size: 1.5em;
            }

            &:hover {
                background: #fff;
                color: #040204;
                cursor: pointer;
            }
        }

    }

    .completed-dance {
        display: block;
        width: 100%;
        height: 300px;
        background-image: url('/images/completed_dance.gif');
        background-repeat: no-repeat;
        background-position: center;
    }

    /* Footer */
    footer {
    position: fixed;
    bottom: 0;
    z-index: 10;
    width: 100%;
    padding: 5px 10px;
    box-sizing: border-box;
    background-color: rgba(255,255,255,0.1);
    font-family: 'Catamaran', sans-serif;
        p  {
            display: inline-block;
            margin: 5px 0;
            font-size: 12px;
            a {
                color: #fff;
            }
        }
    }



    a.donation_link {
        position: fixed;
        font-family: 'Catamaran', sans-serif;
        font-weight: 100;
        padding: 7px 14px;
        background: #0070ba;
        z-index: 10;
        color: #fff;
        top: 20px;
        right: 20px;
        text-decoration: none;
        border-radius: 14px;
        opacity: 0.8;
        transition: all 0.2s;

        i {
            margin: 0 5px;
            position: relative;
            top: 1px;
            transition: all 0.3s;
            
        }
        &:hover {
            opacity: 1;
            i {
                color: red;
            }
        }
    }


    @media screen and (max-width: 700px){
        .beta {
            width: 100%;
            border-radius: 0;
        }
        a.donation_link {
            width: 260px;
            top: 85px;
            right: 50%;
            margin-right: -130px;
            box-sizing: border-box;
        }
        .flex-container .tools ul {
            margin: 20px 0 0 0;
        }
        .flex-container .tools ul li {
            margin-bottom: 20px;
        }
        footer p {
            display: block;
        }
        .generateGoalButton {
            margin-top: 20px;
        }

        /* Completed screen */
        .complete-screen {
            padding: 10px;
        }
    }

.fade-enter-active, .fade-leave-active {
    transition: opacity .6s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
    opacity: 0;
}

</style>

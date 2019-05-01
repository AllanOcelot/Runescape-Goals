<template>
    <div class="page-container">
        <div class="flex-container" v-bind:style="{ backgroundImage: 'url(/images/bg/' + backgroundNumber + '.png)' }">
            <div class="wrapper">
                <div class="beta">
                    Alpha Build
                </div>
                <div class="content">
                    <div class="title">
                        <h1>Runescape Goals</h1>
                        <p>Everything is progress</p>
                    </div>
                    <div class="tools" v-if="generatedGoal === false" >
                        <p>
                        I would like a
                        <span class="button length" v-bind:class="{ active: this.goalLength === 'short' }"
                        v-on:click="toggleLength('short')">
                        Short</span>
                        /
                        <span class="button length" v-bind:class="{ active: this.goalLength === 'long' }"
                        v-on:click="toggleLength('long')">
                        Long</span> term goal.
                        </p>
                        <br>
                        <p>I would like my goal to focus on:
                            <ul>
                                <li>
                                    <span class="button" v-on:click="toggleFocus('quests')"
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
                                    <span class="button" v-on:click="toggleFocus('minigames')"
                                            v-bind:class="{ active: this.goalFocus === 'minigames' }"
                                    >
                                        <img src="/images/icon3.png" alt="Minigame Icon"> Minigames
                                    </span>
                                </li>
                                <li>
                                    <span class="button" v-on:click="toggleFocus('combat')"
                                          v-bind:class="{ active: this.goalFocus === 'combat' }"
                                    >
                                        <img src="/images/icon4.png" alt="Combat icon"> Combat
                                    </span>
                                </li>
                                <li>
                                    <span class="button" v-on:click="toggleFocus('strange')"
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
                    <div class="goal_display" v-else>
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
                </div>
            </div>
        </div>
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
        <footer>
            <p>
                This is an Open Source Project - <a href="https://github.com/AllanOcelot/Runescape-Goals">Contribute Today!</a>
            </p>
            <p>
                Made by <a href="http://allancodes.com" target="_blank">allanCodes</a>.
            </p>
        </footer>
        <a class="donation_link" href="http://www.paypal.me/McKernan" target="_blank"><i class="fas fa-heart"></i> Please consider a donation <i class="fas fa-heart"></i> </a>
    </div>
</template>

<script>
import axios from 'axios';

export default {
        name: 'homescreen',
        props: {
            msg: String
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
                        var task_subtitle      =   self.goalData.data[sub_seed][subset_name].title;
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
<style scoped>

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

    .flex-container .content {
        position: relative;
        color: #fff;
        z-index: 2;
    }


    /* Title */
    .flex-container .title {
        position: relative;
        margin-bottom: 40px;
    }
    .flex-container .title h1 {
        font-family: 'Catamaran', sans-serif;
        padding: 0;
        margin: 0;
        font-size: 2.4em;
    }
    .flex-container .title p {
        position: relative;
        padding: 0;
        margin: 0 auto;
        width: 160px;
    }
    .flex-container .title p::before,
    .flex-container .title p::after {
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
    .flex-container .title p::after {
        left: auto;
        right: -65px;
    }


    /* The goal tools */
    .flex-container .tools p {
        margin: 0 0 10px 0;
        cursor: default;
    }

    .flex-container .tools ul {
        display: block;
        width: 100%;
        text-align: center;
        margin: 15px 0 20px 0;
        padding: 0;
    }

    .flex-container .tools ul li {
        list-style: none;
        display: inline-block;
    }

    .flex-container .tools span.button {
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
    }

    /* Buttons with class working, work */
    .flex-container .tools span.button.working {
        opacity: 1;
    }

    /* Make the length buttons fully visible, as they work  */
    .flex-container .tools span.button.length {
        opacity: 1;
    }

    .flex-container .tools span.button img {
        position: relative;
        top: 2px;
    }

    .flex-container .tools span.button.active,
    .flex-container .tools span.button.active:hover,
    .flex-container .tools span.button:hover{
        padding: 5px 15px;
        margin: 0 5px;
        cursor: pointer;
        border: 1px solid #fff;
        background: #32af41;
        color: #fff;
        border-radius: 4px;
        opacity: 1;
    }

    .flex-container .tools span.button:hover {
        background-color: transparent;
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
    }
    .generateGoalButton:hover {
        cursor: not-allowed;
        opacity: 0.6;
    }

    .generateGoalButton.active {
        background-image: linear-gradient(#67de76, #32af41);
        cursor: pointer;
        opacity: 1;
    }
    .generateGoalButton.active:hover {
        opacity: 1;
        margin-top: 25px;
        margin-bottom: 5px;
        box-shadow: 0px 0 20px rgba(255,255,255,0.6);
    }

    /* The Goal */
    .goal_display {
        padding: 30px 60px;
        background-image: url('/images/goal_bg.png');
        background-size: contain;
        background-repeat: no-repeat;
        background-position: center;
        color: #faff00;
        text-shadow: 2px 2px 0px rgba(0,0,0,0.6);
        width: 400px;
        height: 300px;
        box-sizing: border-box;
        padding: 40px 70px;
        z-index: 20;
    }

    .goal_display h2 {
        display: block;
        width: 100%;
        font-size: 1.6em;
        margin: 0 0 10px 0;
        padding: 0;
        font-family: 'Catamaran', sans-serif;
    }

    .goal_display h3 {
        margin-top: 0;
        text-transform: capitalize;
    }

    .goal_display p {
        margin: 0;
    }

    .goal_display .completed {
        height: 33px;
        line-height: 35px;
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
        background-image: linear-gradient(#ffffff00, #ffffff00);
        transition: all 0.3s;
    }

    .goal_display .completed:hover {
        background-image: linear-gradient(#67de76, #32af41);
        cursor: pointer;
    }

    .goal_display .back {
        position: relative;
        bottom: -8px;
        left: -20px;
        width: 300px;
        font-size: 0.9em;
        text-shadow: none;
        color: #fff;
        cursor: pointer;
    }
    .goal_display .back:hover {
        text-decoration: underline;
    }

    /* Completed screen */
    .completed-screen {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background: #040204;
        z-index: 100;
        padding: 10vh;
        box-sizing: border-box;
        align-items: center;
        -webkit-box-pack: center;
    }


    .completed-screen h1 {
        font-size: 3em;
        letter-spacing: 2px;
    }

    .completed-screen p {
        font-size: 1.3em;
    }

    .completed-screen .close {
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
    }

    .completed-screen .close:hover {
        background: #fff;
        color: #040204;
        cursor: pointer;
    }

    .completed-screen .close i {
        position: relative;
        margin: 0 5px;
        top: 4px;
        font-size: 1.5em;
    }

    .completed-screen .completed-dance {
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
    }

    footer p  {
        display: inline-block;
        margin: 5px 0;
        font-size: 12px;
    }

    footer p a {
        color: #fff;
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
    }

    a.donation_link i {
        margin: 0 5px;
        position: relative;
        top: 1px;
        transition: all 0.3s;
    }

    a.donation_link:hover {
        opacity: 1;
    }

    a.donation_link:hover i {
        color: red;
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

</style>

---
title: "I've Been Writing a 2D Game Engine"
date: 2026-07-15
draft: false
description: "A deep dive into writing a 2D game engine from scratch using SDL2 and C++"
---

For the past few months i have been writing a game engine , truth be told this was a topic that always fascinated me everytime i ventured into computer graphics and rendering , i once attempted to write a vulkan game engine and failed miserably as i dont recall a lot of it now , in hindsight i believe i shouldve made an attempt to be rpesent while making it instead of copying things i didnt undejrstand from other codebases , well the thing about vulkan is that there isnt much to go about , resources are very scarce compared to opengl or sdl2 for that matter but for game engines in general there is plenty to go about , plethora of engines and courses to go about , writing a renderer is a different part but be that as it may lets talk about what i actually want to talk about , game engines , game engines can actually be repurposed onto be anything to be honest but as the name suggests the primary utility is to make games , i ventured to make a 2d game engine , well i did make a throwaway sample one with imgui and sdl2 using c++ was prettry fun but as i said retaining a lot of what i did is really difficult , i have no idea why this is , but writing it out or explaining it to someone like i am doing this with this blog makes a lot of sense to me as i usualyl dont remember doing stuff but i do remember teaching people stuff so this seemed like the perfect way to go about things from here on , i will try to make this sound technical as i probably can but as i write this for myself its really doubtful anyone will ever read this or spend the time to go through as i am neither someone famous nor someone who is good at what he does so its more like a me yapping into the void kinda writing so it may be really informal with a lot of grammatical mistakes but nevermind onto the interesting parts

so what is a game engine and how do they work ?
well game engines are as we said tools we use to make games , we can think of 2 types which are 2d and 3d , 2d game engines are quite easier to work with or make for the most parts as you dont have to deal with vulkan or opengl apis and just use sdl2 or in theory you could use whatvere you want to make a 2d engine , but in principle its easier to use something like sdl2 as it works just fine , in a naive way to think game engines basically do 3 things which actually comprises together to be the game loop

and what is a game loop ?
a game loop consists of 3 parts 4 if you count the destruction of the loop ,
first we process the input
then we update
then we render
then we loop
then we destroy whenever its needed
thats about it for the game loop

![game loop diagram](/images/game-loop.png)

if you think about it you can represent it in code terms with

`while(true){`
`game -> processInput();`
`game -> update();`
`game->render();`
`}`

so like we said we have these three things to do while our game is running
but what do we do when we wanna exit it ?

well thats kinda great cause sdl already provides what we need for this with SDL_QUIT
or in a more keyboard end user way with SDL_KEYDOWN and assigning keydown to escpae key or whatever you want tbh
so we know how the game loop structure looks and how to exit the loop so i guess we can look at each of those functions in a closer light now

First lets talk about initalizing the window before getting into the game loop and how each of those functions work , for the game loop to render onto our screen and does the things the game loop is supposed to do we need to see the changes visually for this we need to initialize our window , well thats great cause sdl can help us handle this too we do SDL_CREATEWINDOW and we got windows now but it does accept a lot of parameters first is called as the title which is the title of the window we are creating then is the x and y cordinated of the window , we can even set this to sdl_windowpos_centered for both x and y then sdl will automatically figure out based on our screen resolution where to place the window exactly then we have width and height which is used to make the windows bigger or smaller as to our liking and the last parameter is flags , flags are kinda useful especially for something like this so we can design our window to look exactly how we want for eg there is a flag to make the window borderless and we do a little pattern matching for errors like if window isnt created error out with std::cerr error creating sdl window etc and then we just init the sdl renderr with sdl_create renderer so that we intialize the renderer along wtih the window , sdl create renderer also accepts some parameters whcih are index , index stands for the monitor or display the content or the window is being displayed to , we usually use -1 which means the default one available and the next one is window which is the window through which we want the renderer to be created through and the last is as usual flags for the rendererand thats about it for creating windows

Beginning with the gameloop functions we have ProcessInput, processing input means as the name suggests processing inputs of whatever the end user is trying to do , in some cases it might be to go up or go down or even as we said to exit the game and as we earlier explored we can use the sdl keydown to assign whatver we want to this specific thing but for this all to work , ok so the user clicked something but how do we know if he clicked something ? we use something from sdl called sdl_event and sdl_poll event , for getting the inputs first we create an sdl event , then we use that event and poll for the keydowns happening in that event , below is an example of this in action

`void Game::ProcessInput() {`
  `SDL_Event event;`
  `SDL_PollEvent(&event);`
  `switch (event.type) {`
  `case SDL_KEYDOWN: {`
    `if (event.key.keysym.sym == SDLK_ESCAPE) {`
      `isRunning = false;`
    `}`
  `}`

Up next we have update function , ok so cool we got the processed input from the user now what should we do ? we move onto update this to our game loop inorder to render this to our screen , updating gets a little bit tricky if we think beyond the naive way of how update works , on paper we just have to get whatever is the value and just update it so our renderer can just go and draw the updated values , to deviate for a moment lets think if we are drawing some projectile onto our screen , and its supposed to move in our screen , the only issue with this is this acts differently in different machines based on the perfomance of our machine , well we dont want that to happen to use right for this we use frame target time and delta time , frame target time is the time taken by a frame to render onto the screen and fps as many of us know is frames per second , so in our update function we use this deltatime to ensure that its fixed or we clip this properly for everyone we can do this in 2 ways either just use a while loop or use something from sdl called sdl delay , the while loop is a very naive way to go about this as we all know while loops are cpu heavy processes and we shouldnt really spam it everywhere unless we really need them we could just use

`while(!SDL_TICKS_PASSED(SDL_GetTicks(), TicksLastFrame + FRAME_TARGET_TIME));`

but why do that when we have sdl_delay in our hands so we use that instead

up next we have render , yaay we get to render things onto our screen now , the way renderer for sdl2 works is that we have something called double buffering , which means that we have 2 buffers front and back buffers and in principle we first we set sdl renderer draw color then we swap the front nad back buffers with SDL_RenderPresent , setrenderdrawcolor which is the fucntion that draws the colors accepts 4 arguments which are the renderere where we want to draw and rgba , rgba is basically red green blue and a for alpha or opacity and finalyl we destroy whatever is drawn onto the screen

commit -> https://github.com/loveucifer/eclipse/commit/72b6bc7d2de1bb89c7fa28cbcf057b9c570fdbd9
engine -> https://github.com/loveucifer/eclipse

this was deeply influenced by a lot of tutorials and books i saw on game engines and a lot of open source codes i read , so i doubt any of this is my actual thoughts in order for me to say it is but this is in a way me explaining stuff to myself so i dont forget what i do like always.

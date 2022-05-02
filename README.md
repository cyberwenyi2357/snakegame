# CS2303 Course Project - Snake Game

## The Snake Game

### Introduction
"Snake" is a classic video game,
where the player controls a snake to pick up food and avoid running into blocks or its own body.
In our Snake Game, there are no "walls" surrounding the playing area,
and whenever the snake goes beyond the boundary,
it will come out of the opposite side.
(for example, if it reaches the rightmost boundary, it will re-emerge from the leftmost one.)
Each time the snake eats a piece of food, its tail grows longer by one segment,
and its speed also increases.
When the snake runs into a roadblock (the dark green blocks),
or a segment of its own body, it dies.

Our Snake Game also supports `Undo` after the snake dies.
An Undo command (by pressing `Backspace`) will change the state of the snake back to the last time it eats a piece of food.
Afterwards, the player is able to resume the game by pressing `Enter`,
and five new extra roadblocks will be generated.
Each time the snake dies of running into roadblocks,
the roadblock hit by the snake will be removed after the game resumes.
It is also worth noting that the snake body of a previous state might overlap a newly added roadblock.
Therefore, we will also remove all overlapping roadblocks upon resuming the game.


### Instructions
+ Press `Arrow Keys` to control the snake.
+ When the snake dies, press `Backspace` to undo.
+ After the Undo command, press `Enter` to resume the game.


**note:** _please refer to `SnakeGame.mp4` for complete functionality of the game._
 
***

## Implementation
There are three compulsory coding tasks,
and two optional tasks from which you should choose one.
For each item of the TO-DO checklist in three compulsory coding tasks,
please specify the following in your report:
- Explanation of your code
- An analysis of time complexity 

If any alternative data structure exists,
in your report please compare them with the current one in terms of time and space complexity,
and specify why or why not the current version is preferred.

### Snake Body -- Linked List
In our implementation,
the head of the snake is stored in `head_x` and `head_y` in the `Snake` class,
while the body of the snake is stored in a linked list,
the class of each node defined as the following in `include/snake.h`:
```c++
class SnakeBodySegment {
public:
    SDL_Point point;
    SnakeBodySegment *next;

    SnakeBodySegment(SDL_Point _point)
    {
        point = _point;
        next = nullptr;
    }
};
```
The `(x, y)` coordinate of each `point` can be accessed by `point.x` and `point.y`.

The head of the linked list is the tail of the snake body.
Each time the snake moves to a new cell,
we insert the new cell to the tail of the linked list.
In the meantime, if the snake does not grow longer,
we remove the head of the linked list (which is the snake tail).
All other segments of the snake body remain in the same place.
The head and tail node of the linked list are stored in `SnakeBodySegment* head`
and `SnakeBodySegment* tail` in the `Snake` class respectively.

However, as you might notice,
the snake currently does not grow any longer whenever it picks up food.
This is because the code for inserting the new cell is incomplete,
and the renderer also neglects the snake body.

**TO-DO**
- [ ] Please implement inserting new cells to the linked list.
(`Snake::UpdateBody` in `src/snake.cpp`)
- [ ] Please implement rendering of the snake body.
(`Renderer::Render` in `src/renderer.cpp`)

Please be informed that the `prev_head_cell` argument in `Snake::UpdateBody`
refers to the previous head of the snake, which should be the new cell
inserted to the snake body.
If you have problems concerning the rendering,
please read the code of `Renderer::Render` carefully.
If your code works properly, the snake should be able to grow longer,
and the snake body should be rendered in white.

### Undo -- Stack
In order to implement the Undo Operation,
we make use of the Standard Template Library (STL) in C++.
A stack holding previous states of the snake is defined as
`std::stack<Snake> states` in the `Game` class,
which entails `#include<stack>`.
The stack in C++ STL supports all basic operations such as
`push`, `pop`, `top`, `size`, `empty`, etc.
Please refer to https://www.geeksforgeeks.org/stack-in-cpp-stl/ for more detailed information.

In order to store states of the snake after it picks up food,
the snake is copied to a temporary variable within `Game::Update` in `src/game.cpp`.
After that, you need to set the temporary copy to be "dead", (i.e., `alive = false`),
and push it into the stack.
Besides, the code in `Controller::Undo` is also incomplete.
You need to reverse states of the snake back to the last time it eats a piece of food
when `Backspace` is pressed,
and resume the game (by reviving the "dead" snake) when `Enter` is pressed.

**TO-DO**
- [ ] Please implement the code for pushing the snake copy into the stack.
(`Game::Update` in `src/game.cpp`)
- [ ] Please implement the code for reversing states of the snake.
(`Controller::Undo` in `src/controller.cpp`)
- [ ] Please implement the code for resuming the game.
(`Controller::Undo` in `src/controller.cpp`)

### Collision Detection -- Quadtree
We use Quadtree as the data structure to hold all roadblocks in the game.
The class of Quadtree node is defined as follows, in `include/roadblock_node.h`.

```c++
class RoadBlockNode {
public:
    int count{0};
    int left, right;
    int floor, ceiling;

    RoadBlockNode* upper_left{nullptr};
    RoadBlockNode* upper_right{nullptr};
    RoadBlockNode* lower_left{nullptr};
    RoadBlockNode* lower_right{nullptr};

    void UpdateCount();

    RoadBlockNode(int grid_width, int grid_height);

    RoadBlockNode(int left, int right, int floor, int ceiling):
            left(left),
            right(right),
            floor(floor),
            ceiling(ceiling) {}
};
```

Each node of the Quadtree holds the number of roadblocks in `count` within a rectangular region.
The X-axis range of the rectangular region is `[left, right)`, Y-axis `[floor, ceiling)`.
Each node has four children responsible for four subregions
(upper left, upper right, lower left, lower right).
The X-axis of the region is divided into
`[left, (left+right)/2 )` and `[(left+right)/2, right)`;
Y-axis into `[floor, (floor+ceiling)/2 )` and `[(floor+ceiling)/2, ceiling)`.

As you may find out,
although all roadblocks are properly generated and rendered,
there is however no collision detection right now.
The snake just runs over those roadblocks alive.
In order to prohibit the snake from hitting roadblocks,
you need to implement collision detection in `Game::DetectCollision`.
The function takes as input a Quadtree node,
and coordinates of a block in `x` and `y`,
and outputs whether a collision occurs
(i.e., whether there is a roadblock in `(x, y)`).

**TO-DO**
- [ ] Please implement the code for collision detection. (`Game::DetectCollision` in `src/game.cpp`)

If you have difficulties, please refer to `Game::DeleteBlock` and `Game::InsertBlock` for insights.

### Automatic Navigation (Optional)
You are required to navigate the snake to pick up food automatically,
without running into roadblocks or any segment of its body.
You can either choose to implement the code of the algorithm,
or elaborate on your algorithm in your report.
In the report, you should:
- Describe your algorithm in detail.
- Analyse space and time complexity.
- If any additional data structure is needed, please specify.

### Bug Fixing (Optional)
There are potential bugs in the code.
If you happen to find them, please specify the following in your report:
- Why it is a bug instead of an intended behaviour?
- Why this bug occurs?
- How to fix it?

If possible, please implement the code for fixing these bugs.

***

## Submission
__1. The code project folder (Please compress into .zip file)__

__2. Project report (.pdf or .docx file)__

The course project can be done individually or by a group of 2 people.
If it is done by a group, only one person needs to do the submission,
but please describe in your report the detailed contribution of each group member.


## Grading
__Snake Body (Linked List)__ -- 20%

__Undo (Stack)__ -- 20%

__Collision Detection (Quadtree)__ -- 30%

__The Optional Task__ -- 30%
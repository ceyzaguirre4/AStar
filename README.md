#A*/Dijkstra Pathfinding

My own take on A*/Dijkstra Pathfinding. Its a generalization of the typical process so as to permit any posible implementation while using whatever data-structures and heuristic functions regardless of whether or not the "map" fits in memory.


### Overview:

`path = a(<calculate_posibles_func>, <start>, <basecase>, [heuristic=None], [time_limit=0], [reverse=False]) `

This function is designed to solve all pathfinding problems regardless of whether the map/graph/etc fits in memory, or the type of each step (but it must be comparable and hashable). In order to achieve this *"one size fits all"* solution, control over the heuristics, posible moves for any given position and start point are delegated to the user and integrated into the algorithm. They are passed into the funtion as parameters.

The bare minimums for a path to be found are:

`calculate_posibles_func`: Must return a list of tuples `(next_move, move_cost)` where  `move_cost` must be of type int or float, but `next_move` can be of any type. In order to find all valid  `next_move`'s the function must receive a parameter of the same type as  `next_move` representing the current position of the algorithm.

`start `: must be of identical type as  `next_move`. It indicates the starting point of the algorithm, not necesarily the starting point of the map (for example, it is more eficient to find the way from the end to the start but more on that later).

`basecase `: must be a function that only returns `True` when the path arives at its intended destination. For example, in a maze solving problem starting at the entrance,  `basecase` should only return `True` when the current position is the exit of the maze. The function must receive the current position (of whatever type the user decided to create his moves).

The `a()` function returns a list with all the necesary next steps (where the steps are once again of the same type as `next_move`) in order to reach the basecase.

More granular control can be had by passing into `a()`:

`heuristic `: a function that returns a integer or float that represents the likelyhood of a step being the correct one. A lower value means that the step is more likely to be correct than another with a higher `heuristic` return value. It must receive one parameter to indicate the position who's value is to be computed. The one diference between Dijkstra and A* is the inclusion of this function.

`time_limit`: a integer or float that indicates for how long (in seconds) the algorithm should run. Its default value is `0` which permits it to run indefinetely.

`reverse`: a booean that tells the algorithm whether or not it is computing the steps from *"back to front"* (end to begining, instead of the more traditional begining to end). Setting this value to `True` speeds up the extraction of the answer in `a()` by returning a generator object instead of a list. This is generally better if the length of the answer doesnt fit in its allocated memory or if the user doesnt need to know the full path all at once, but rather just the first n-steps that bring it closer to the endpoint (for example, in a game where the path will be re-computed every few miliseconds to consider terrain changes). If set to `True` both `start` and `basecase` have to change to reflect the new direction, and sometimes `calculate_posibles_func ` also has to change (eg: if the map is a directed graph (some directions are only permited one-way), or the `move_cost ` is diferent (uphill vs downhill)).

#### Example code: Maze solving with tuples

This example shows how to compute paths both in normal and in *back-to-front* ways (respectively). The *steps* are tuples (hashable out of the box). The first case uses Dijkstra and the second uses A* with the manhattan distance as its heuristic function.

~~~python
from A import a

# labrynth is represented by a graph of valid moves.
labrynth = [(0,0), (1,0), (1,1), (1,2), (2,0), (3, 0), (3, 1), (4, 1), (4, 2), (4, 3), (3, 3), (3, 4), (3,5), (3, 2)]
start = (0, 0)
end = (3, 2)

#############################
# Some posible heuristics:
#############################

def pythagorean_distance(position):		# heuristic n1
	return (abs(position[0]-start[0])**2 + abs(position[1]-start[1])**2)**0.5

def manhattan_distance(position):		# heuristic n2
	return abs(position[0]-start[0]) + abs(position[1]-start[1])

#############################
# Calculate posibles (identical for both directions because labrynth is a undirected graph and move_cost is always 1)
#############################

def calculate_posibles(position):		# returns valid moves for a given position
	ret = []
	for valid_move in labrynth: 
		if (valid_move[0] == position[0] and valid_move[1] == position[1] + 1) or (valid_move[0] == position[0] and valid_move[1] == position[1] - 1) or (valid_move[0] == position[0] + 1 and valid_move[1] == position[1]) or (valid_move[0] == position[0] - 1 and valid_move[1] == position[1]): 
			ret.append((valid_move, 1)) 	# tuple contains element and movement cost
	return ret

#############################
# Starting from begining:
#############################

def basecase1(position):	# basecase starting from the begining
	if position == end:
		return True
	return False

answer1 = a(calculate_posibles, start, basecase1)  # from begining to end, returns list (no heuristic:Dijkstra; no time limit)

#############################
# Starting from the end:
#############################

def basecase2(position):		# basecase starting from the end
	if position == start:
		return True
	return False

answer2 = a(calculate_posibles, end, basecase2, heuristic=manhattan_distance, reverse=True)  # from back to front, returns generator (includes heuristic cost function: A*)
~~~

#### Example code: Maze solving with user defined types

This example builds on the previous one, only changing the *step-type* (from tuples to a user defined object). All functions remain the same, including the call to `a()`. The *step-type* must be comparable (include `__cmp__()` or `__eq__()`) and hashable (include `__hash__()`). Both functions will be used to check if a position was already checked, so they must be implemented in a way that is:

1. consistent with the users map
2. the comparison must return `True` for the same position in the map
3. objects which compare equal have the same hash value

In the following example two `Node` objects are equal if their x, y coordinates are the same (same position in the map), and two objects which are in the same position will always have the same hash value because the coordinates determine the hash value (so equal coordinates -> equal hash value).

`__getitem__` was implemented to recycle the previous examples code, but is not necesary.

~~~python
class Node:
	def __init__(self, x, y):
		self.coordinates = x, y

	def __eq__(self, other):
		if self.coordinates == other.coordinates:	# equivalent to checking self.x, self.y == other.x, other.y
			return True
		return False

	def __hash__(self):
		return hash(self.coordinates)

	def __getitem__(self,key):
		return self. self.coordinates[key]

# labrynth is represented by a graph of valid moves.
labrynth = [Node(0,0), Node(1,0), Node(1,1), Node(1,2), Node(2,0), Node(3, 0), Node(3, 1), Node(4, 1), Node(4, 2), Node(4, 3), Node(3, 3), Node(3, 4), Node(3,5), Node(3, 2)]
start = Node(0, 0)
end = Node(3, 2)
~~~

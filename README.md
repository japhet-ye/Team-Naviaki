# Team-Naviaki
Source code for the autonomy algorithm for Team Navikai Robosub 

# initial high level source code
// pseudo code
main(){
	// this is how we are going to keep track of time
	// we are just counting how much time has elapsed
	time = initialize_time();

	// we set the target list to an empty list
	// we hardcode in the initialization step so to keep main clear of clutter
	targetList = [];

	// we set the target_queue to an empty queue
	target_queue = [];

	// calling initialization set
	// it will run through the process of making sure the sub is in working order
	// and initialize the targetList and target_queue with the objects we programmed
	Initialization(&targetList,&target_queue);
	
	// main loop
	// the loop will break when 
	// 1) we run out of time or
	// 2) we run out of targets
	while(time <= 20min. and targetList is not empty){

		// target queue will be initialized
		// so we grab the object that is in the front of the queue
		current_target = target_queue.top();

		// deleting that object from the queue
		target_queue.pop();

		// this is where parallel processing comes into play
		// this is the child's directions
		if(fork() == 0){

			// the child will look for targets
			// it will loop through the target list 
			// it will break until 
			// 1) it has found all of the targets or
			// 2) the parent kills the process
			while(target_list is not empty){

				// will call the funtion LoorForTargets
				next_target = LookForTargets(&targetList);

				// if it has found a target, then it will return an object
				// else it will return null
				if(next_target != NULL){

					// if we have found a target, then we push it onto the queue		
					target_queue.push(next_target);
				}
			// loop back up
			}
		}
		// this is the parent process
		else{
			// if navigation is successful
			// which means we are in the optimal position for the target
			// as in face on
			if(NavigateToTarget(current_target)){

				// stop the parallel process
				killChild();

				// then we will do the associated moves that coresspond to the target
				ProcessCommands(current_target);
			}
			else{
				// if naviagtion was not successful
				// still, we kill the parallel process
				killChild();
			}
			// regardless of if we we able to go to the target
			// we must keep searching for more
			// we check to see if we have found all the objects that we are awy of
			if(target_list is empty){

				// is we have, then we stop the run
				// and to do this, we surface
				surface();
			}
			// if we get here, that means there are still targets that we can look for
			// we must then see if we saw something on the way	
			if(target_queue is empty){

				// if we did not, then we start searching for the next target
				// and place it into the target queue
				target_queue.push(LookForTargets(&targetList));
			}		
			
		}
	// loop back up
	}
	// we are at the end of the run, therefore we need to surface
	surface();	
}

// this is the initialization method
// it will do the startup routine
// and initialize the targetList and target_queue
Initialization(&targetList,&target_queue){

	// startupChecks for the sub
	// like make sure that all the sensors are up
	// and make sure that all the actuators are working
	startupChecks();

	// this is where we initialize the lists
	// we can hardcode this step to the desired objects
	// for instance, we know that the first object that we will
	// encounter is the gate
	// so we will hardcode the gate object into the target_queue
	targetList = [list of targets]
	target_queue = [first_target]
}

// this method looks for targets in the target list
LookForTargets(&targetList){

	// we depend on computer vision to see the targets
	// this will return an array of objects tied to the confidence of each object
	// IOW, [<gate, 0.9>, <lever, 0.03>, <path, 0.08>] 
	computer_vision_results = computerVision(camera);

	// we want to sort by the object with the highest confidence
	// as in the object that is closest
	best_result = max(computer_vision_results);

	// we then check to see if we actually need to go to the target
	// if we do not need to go to the target, then we keep looping
	// through the computer_vision_results until we get the object that we need to do
	while(targetList.find(best_result) == NULL){

		// deleting the object from the reults list
		computer_vision_result.remove(best_result);

		// set best_result to the new maximum
		best_result = max(computer_vision_results);
	}
	// return the object
	return best_result;

}

// this is the navigation method
NavigateToTarget(current_target){

	// we base it off the confidence that the computerVision software returns
	computer_vision_confidence = computerVision(camera).look(current_target);

	// we set a threshold to know to how confident that we want
	// this loop determines if we see the object that we want
	while(computer_vision_results < 85){

		// if we are not that confident in the object we are seeing
		// we now need move in the direction that will make
		// our confidence go up
		// delta is initially 0
		delta = 0;	

		// we will try different manuevers (i.e forward, rotate, etc.)
		// and see if our confidence is increasing
		// the loop will break if the confidence goes up		
		while(delta <= 0){

			// we try different moves and saving the move
			current_move = trymoves();
			actuator.move(current_move);

			// calculation of delta
			// delta > 0 will break the loop
			// we compare the previous confidence with the current confidence after the move
			delta = computer_vision_confidence - computerVision(camera).look(current_target);

		// loop back up
		}

		// at this point we know that the move we made will increase our confidence
		// therefore we do this move until we reach the desired confidence
		actuator.move(current_move);

	// loop back up
	}
	// return that we are successful
	return 1;
}
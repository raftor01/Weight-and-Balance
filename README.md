# Weight-and-Balance
import java.util.InputMismatchException;
import java.util.Scanner;

public class WnB {
	static Scanner keyboard = new Scanner(System.in);
	static double ACweights[] = {1691.6, 1654.7, 1660.2, 1701.6, 1655.5}; //0:FSTP, 1: GMRU 2: GLJL 3: GEKA 4: GTBF 
	static double ACmoments[] = {69817, 67249, 67249, 71082, 67233}; //0:FSTP, 1: GMRU 2: GLJL 3: GEKA 4: GTBF 
	static double ACweightsWF[] = {1692.4, 1660.6, 1661.0, 1702.4, 1656.3}; //0:FSTP, 1: GMRU 2: GLJL 3: GEKA 4: GTBF 
	static double ACmomentsWF[] = {69800, 67112, 67232, 71066, 67216}; //0:FSTP, 1: GMRU 2: GLJL 3: GEKA 4: GTBF 
	
	private static void run(){
		String ident;
		double burn[] = {0,0,0}; // 0: Mission time, 1: Planned consumption, 2:MRUreserve
		double weights[] = {0,0,0,0,0,0}; //0: Front, 1: Rear, 2: Baggage 1, 3: Baggage 2, 4: Fuel, 5:Empty/total
		double moments[] = {0,0,0,0,0,0}; //0: Front, 1: Rear, 2: Baggage 1, 3: Baggage 2, 4: Fuel, 5:Empty/total
		welcomeMessage();
		showDisclaimer();
		promptWeightTable();
		promptEquipTable();
		ident = promptIdent();
		boolean winterFronts = winterOps(ident);
		weights[5] = getEmptyWeight(ident, winterFronts);
		moments[5] = getEmptyMoment(ident, winterFronts);
		fillWeights(weights);
		calculateMoments(moments, weights);
		fillTheBurn(burn, weights);
		printReport(weights, moments, burn, ident);
		promptEnd();
		
		keyboard.close();
	}
	
	private static void welcomeMessage(){
		System.out.println("Mount Royal Aviation Aircraft Weight and Balance Calculator v4.0.2"); 
		System.out.println("Created by Rafael Acuna");
		System.out.println("Weights updated February 1, 2021");
		//Version History:
		//4.0.2 GLJL, GEKA WFWeight updated
		//4.0.1 Formatting 
		//		Removed Jyothi Bachwala, Marise Meiklejohn from instructr weight list
		//		Added Chrissy Perry (need actual weight) to instructor weight list
		//		Updated weight for Joss Engen, Keagan Vaz, Rafael Acuna
		//4.0 Added checks for entering utility before landing, and entering or going past reserve fuel
		//3.9 Updated basic empty weights for all A/C. Moved values to double arrays at top of file. Updated equipment and instructor weights. 
		//		Removed Joe Wagner and added Brett Richards. 
		//3.8.3 Updated basic empty weights for with and without winter fronts
		//3.8.2 Added Labels to final printout to make it more text-to-speech friendly, added weight value for Jyothi Bachwala
		//3.8.1 Updated FSTP weights as per skid plate installation. Changed "Weights current...." to "Weights updated..."
		//3.8 Updated Instructor and aircraft item weights, added 2 lbs of permanent baggage in front seats for FAK, and 2 lbs in baggage 2 for tow bar. Winter fronts weights not verified as of this version.
		//3.7 Made Utility/Normal prompt more consistent with rest of program
		//3.6 Alphabetized Instructor Names, checked weights
		//3.5 Fixed crash when a mathematical operation was entered in some fields. Formatting changes.
		//3.4 Updated GMRU as a spin plane, removed Chris Finot from instructor weight list
		//3.3 Updated winter front weight and moment database for GTBF and GMRU, formatting changes, removed debug code
		//3.2 Fixed Max T/O weight checking 
		//3.1 Updated Instr weight table, Added Equipment table
		//3.0 Added winter fronts prompt and appropriate weight and balance numbers, implemented restart function. testing stripped down restart.
		//2.6 Removed printing "bug" on screen
		
			
	}
	private static boolean showDisclaimer(){
		String response = null;
		boolean loop = true;
		boolean accept = false;
		System.out.println("\nThis program is intended to aid in flight planning. \nIt is NOT meant to supersede weight and balance calculations as per the aircraft flight manual. \nThe PIC is fully responsible for ensuring the aircraft is operated within limits.\n");
		while(loop){
			System.out.print("Do you agree to these terms and conditions? (Y/N) ");
			response = keyboard.next();
			if(response.equalsIgnoreCase("Y")){
				loop = false;
				accept = true;
			}
			else if(response.equalsIgnoreCase("N")){
				System.out.println("You must accept the terms in order to use this program. Thank you.");
				promptExit();
			}
			else{
				System.out.println("Error. Invalid input. Please enter either (Y/N).\n");
			}
		}
		System.out.println("---------------------------------------------------------------------------------------------------------");

		return accept;
	}
	private static void promptWeightTable(){
		String response;
		boolean loop = true;
		while(loop){
			System.out.print("Do you need the instructor weight table? (Y/N) ");
			response = keyboard.next();
			if(response.equalsIgnoreCase("Y")){
				printWeightTable();
				loop = false;
			}
			else if(response.equalsIgnoreCase("N")){
				loop = false;
			}
			else {
				System.out.println("Error. Invalid input. Please enter either (Y/N).\n");
			}
		}
	}
	private static void printWeightTable(){
		System.out.println("\nInstructor Weights (lbs):\n");
		System.out.println("Acuna, Rafael..........175");
		System.out.println("Aragon-Franssen, Juan..200");
		System.out.println("Coleman, Steve.........250");
		System.out.println("Engen, Joss............200");
		System.out.println("Farr, Colton...........165");
		System.out.println("Kitzul, Jacob..........145");
		System.out.println("Lee, Juno..............160");
		System.out.println("Perry, Chrissy.........130");
		System.out.println("Richards, Brett........155");
		System.out.println("Rooke, Norm............210");
		System.out.println("Stooke, Zac............170");
		System.out.println("Vaz, Keagan............165");
		System.out.println("Weibe, Deanna..........165");
		System.out.println("---------------------------------------------------------------------------------------------------------");
	}
	private static void promptEquipTable(){
		String response;
		boolean loop = true;
		while(loop){
			System.out.print("Do you need the equipment weight table? (Y/N) ");
			response = keyboard.next();
			if(response.equalsIgnoreCase("Y")){
				printEquipTable();
				loop = false;
			}
			else if(response.equalsIgnoreCase("N")){
				loop = false;
			}
			else {
				System.out.println("Error. Invalid Input. Please try again.\n");
			}
		}
	}
	private static void printEquipTable(){
		System.out.println("Sleeping Bags (2)..............11");
		System.out.println("Aft Baggage Bin................18");
		System.out.println("Rear Bag (spin planes)..........3");
		System.out.println("Tow Bar.........................2");
		System.out.println("---------------------------------------------------------------------------------------------------------");

	}
	private static void promptExit(){
		String response;
		boolean loop = true;
		while(loop){
			System.out.print("Would you like to exit? (Y/N) ");
			response = keyboard.next();
			if(response.equalsIgnoreCase("Y")){
				System.out.println("Exiting...");
				System.exit(0);
			}
			else if(response.equalsIgnoreCase("N")){
				loop = false;
			}
			else{
				System.out.println("Invalid input. Please try again.\n");
			}
		}
	}
	private static String promptIdent(){
		String input = null;
		boolean ident = false;
		while(!ident){
			System.out.print("Please enter the ident of the aircraft: ");
			input = keyboard.next();						//test validity of ident
			ident = validIdent(input); 						//set boolean
			if(!ident){
				System.out.println("Error. Invalid aircraft ident. Please try again.\n");
			}
		}
		return input;
	}
	private static boolean winterOps(String ident){
		boolean winterFronts = false;
		String response;
		boolean loop = true;
		while (loop){
			System.out.print("Will C-" + ident.toUpperCase() + " be operated with winter fronts installed? (Y/N) ");
			response = keyboard.next();
			if(response.equalsIgnoreCase("Y")){
				winterFronts = true;
				loop = false;
			}
			else if(response.equalsIgnoreCase("N")){
				winterFronts = false;
				loop = false;
			}
			else{
				System.out.println("Invalid input. Please try again\n");
			}
		}
		System.out.print("---------------------------------------------------------------------------------------------------------");
		return winterFronts;
	}
	private static boolean validIdent(String ident){
		if(ident.equalsIgnoreCase("FSTP")){
			return true;
		}
		if(ident.equalsIgnoreCase("GMRU")){
			return true;
		}
		if(ident.equalsIgnoreCase("GLJL")){
			return true;
		}
		if(ident.equalsIgnoreCase("GEKA")){
			return true;
		}
		if(ident.equalsIgnoreCase("GTBF")){
			return true;
		}
		return false;
	}
	private static double getEmptyWeight(String ident, boolean winterOps){		
		if(ident.equalsIgnoreCase("FSTP") && !winterOps){
			return ACweights[0];
		}
		if(ident.equalsIgnoreCase("GMRU") && !winterOps){
			return ACweights[1];
		}
		if(ident.equalsIgnoreCase("GLJL") && !winterOps){
			return ACweights[2];
		}
		if(ident.equalsIgnoreCase("GEKA") && !winterOps){
			return ACweights[3];
		}
		if(ident.equalsIgnoreCase("GTBF") && !winterOps){
			return ACweights[4];
		}
///////////////////////////////////////////////////////////////
		if(ident.equalsIgnoreCase("FSTP") && winterOps){
			return ACweightsWF[0];
		}
		if(ident.equalsIgnoreCase("GMRU") && winterOps){
			return ACweightsWF[1];
		}
		if(ident.equalsIgnoreCase("GLJL") && winterOps){
			return ACweightsWF[2];
		}
		if(ident.equalsIgnoreCase("GEKA") && winterOps){
			return ACweightsWF[3];
		}
		if(ident.equalsIgnoreCase("GTBF") && winterOps){
			return ACweightsWF[4];
		}
		return -1;
	}
	private static void fillWeights(double[] weights){
			System.out.println();
			boolean valid = false;
			while(!valid){
				try{
					System.out.print("Please enter the weight of the PIC: ");
					weights[0] = Double.parseDouble(keyboard.next());
					if ((weights[0] >0)){
						valid = true;
					}
					else{
						System.out.println("Error. Value cannot be zero or negative. Please try again.\n");
					}
				}
				catch(InputMismatchException e){
					System.out.println("Error. Invalid input. Please try again.\n");
									}
				catch(NumberFormatException e){
					System.out.println("Error. Invalid input. Please try again.\n");
				}
			}
			valid = false;
			while(!valid){
				try{
					System.out.print("Please enter the weight of the front passenger (if none, enter 0): ");
					double input = Double.parseDouble(keyboard.next());
					weights[0] = weights[0] + input + 2; //add 2 lbs each time for FAK.  
					if ((input >=0)){
						valid = true;
					}
					else{
						System.out.println("Error. Value cannot be negative. Please try again.\n");
					}
				}
				catch(InputMismatchException e){
					System.out.println("Error. Invalid input. Please try again.\n");
				}
				catch(NumberFormatException e){
					System.out.println("Error. Invalid input. Please try again.\n");
				}
			}
			valid = false;
			while(!valid){
				try{
					System.out.print("Please enter the weight of the rear passenger 1 (if none, enter 0): ");
					weights[1] = Double.parseDouble(keyboard.next());
					if ((weights[1] >= 0)){
						valid = true;
					}
					else{
						System.out.println("Error. Value cannot be negative. Please try again.\n");
					}
				}
				catch(InputMismatchException e){
					System.out.println("Error. Invalid input. Please try again.\n");
					}
				catch(NumberFormatException e) {
					System.out.println("Error. Invalid input. Please try again.\n");
			}
			valid = false;
			while(!valid){
				try{
					System.out.print("Please enter the weight of the rear passenger 2 (if none, enter 0): ");
					double inpuT = Double.parseDouble(keyboard.next());
					weights[1] = weights[1] + inpuT;
					if ((inpuT >= 0)){
						valid = true;
					}
					else{
						System.out.println("Error. Value cannot be negative. Please try again.\n");
					}
				}
				catch(InputMismatchException e){
					System.out.println("Error. Invalid input. Please try again.\n");
					}
				catch(NumberFormatException e){
					System.out.println("Error. Invalid input. Please try again.\n");
				}
			}
			valid = false;
			while(!valid){
				try{
					System.out.print("Please enter the weight of the item(s) in Baggage Area 1: ");
					weights[2] = Double.parseDouble(keyboard.next());
					if ((weights[2] >= 0)){
						valid = true;
					}
					else{
						System.out.println("Error. Value cannot be negative. Please try again.\n");
					}
				}
				catch(InputMismatchException e){
					System.out.println("Error. Invalid input. Please try again.\n");
					}
				catch(NumberFormatException e){
					System.out.println("Error. Invalid input. Please try again.\n");
				}
			}
			valid = false;
			while(!valid){
				try{
					System.out.print("Please enter the weight of the item(s) in Baggage Area 2: ");
					weights[3] = Double.parseDouble(keyboard.next()) + 2; //2 lbs each time for tow bar
					if ((weights[2] >= 0)){
						valid = true;
					}
					else{
						System.out.println("Error. Value cannot be negative. Please try again.\n");
					}
				}
				catch(InputMismatchException e){
					System.out.println("Error. Invalid input. Please try again.\n");
					}
				catch(NumberFormatException e){
					System.out.println("Error. Invalid input. Please try again.\n");
				}
			}
			valid = false;
			while(!valid){
				try{
					System.out.print("Please enter the fuel on board (in gallons): ");
					weights[4] = 6 * Double.parseDouble(keyboard.next());
					if ((weights[4] > 0)){
						valid = true;
					}
					else if(weights[4] <= 0){
						System.out.println("Error. Value cannot be zero or negative. Please try again.\n");
					}
					if(weights[4] / 6 > 53){
						valid = false;
						System.out.println("Error. Maximum fuel exceeded. Please try again.\n");
					}
				}
				catch(InputMismatchException e){
					System.out.println("Error. Invalid input. Please try again.\n");
				}
				catch(NumberFormatException e){
					System.out.println("Error. Invalid input. Please try again.\n");
				}
			
			}
		}
	}
	private static double getEmptyMoment(String ident, boolean winterOps){
		if(ident.equalsIgnoreCase("FSTP") && !winterOps){
			return ACmoments[0];
		}
		if(ident.equalsIgnoreCase("GMRU") && !winterOps){
			return ACmoments[1];
		}
		if(ident.equalsIgnoreCase("GLJL") && !winterOps){
			return ACmoments[2];
		}
		if(ident.equalsIgnoreCase("GEKA") && !winterOps){
			return ACmoments[3];
		}
		if(ident.equalsIgnoreCase("GTBF") && !winterOps){
			return ACmoments[4];
		}
///////////////////////////////////////////////////////////////
		if(ident.equalsIgnoreCase("FSTP") && winterOps){
			return ACmomentsWF[0];
		}
		if(ident.equalsIgnoreCase("GMRU") && winterOps){
			return ACmomentsWF[1];
		}
		if(ident.equalsIgnoreCase("GLJL") && winterOps){
			return ACmomentsWF[2];
		}
		if(ident.equalsIgnoreCase("GEKA") && winterOps){
			return ACmomentsWF[3];
		}
		if(ident.equalsIgnoreCase("GTBF") && winterOps){
			return ACmomentsWF[4];
		}
		return -1;
	}
	private static void calculateMoments(double[] moments, double[] weights){
		int[] arms = {37, 73, 95, 123, 48}; //0: Front, 1: Rear, 2: Baggage 1, 3: Baggage 2, 4: Fuel
		for(int i = 0; i <= 4; i++){
			moments[i] = weights[i] * arms[i]; //Find moments of each 
		}
	}
	private static void zeroFuel(double[] weights, double[] moments){
		for(int i = 0; i <= 3; i++){
			weights[5] = weights[5] + weights[i];
			moments[5] = moments[5] + moments[i];
		}

	}
	private static void fillTheBurn(double[] burn, double[] weights){
		boolean valid = false;
		while(!valid){
			try{
				System.out.print("Please enter the planned mission time (hours): ");
				burn[0] = Double.parseDouble(keyboard.next());  //Value for time
				if ((burn[0] > 0)){
					valid = true;
				}
				else{
					System.out.println("Error. Value cannot be zero or negative. Please try again.\n");
				}
			}
			catch(InputMismatchException e){
				System.out.println("Error. Invalid value entered. Please try again.\n");
			}
			catch(NumberFormatException e){
				System.out.println("Error. Invalid value entered. Please try again.\n");
			}
		}
		valid = false;
		while(!valid){
			try{
				System.out.print("Please enter the planned fuel burn (GPH): ");
				burn[1] = burn[2] = Double.parseDouble(keyboard.next()); // Value for GPH
				if ((burn[1] > 5.6)){ // checks against minimum fuel burn listed in the R model POH 5-18
					valid = true;
				}
				else{
					System.out.println("Error. Value is lower than the minimum cruise performance of this aircraft. Please try again.\n");
				}
			}
			catch(InputMismatchException e){
				System.out.println("Error. Invalid value entered. Please try again.\n");
			}
			catch(NumberFormatException e){
				System.out.println("Error. Invalid value entered. Please try again.\n");
			}
		}
		burn[0] = burn[0] * 6 * -burn[1]; //converts to fuel burn in lbs, makes it negative
		burn[1] = burn[0] * 48; //calculates moment for fuel burn
		burn[2] = burn[2] * 6; //calculates reserve fuel in lbs
		// now: burn[burnWeight, burnMoment, MRUreserve]
	}
	private static void landingWeight(double[] weights, double[] moments, double[] burn){
		weights[5] = weights[5] + burn[0];
		moments[5] = moments[5] + burn[1];
	}
	private static void rampWeight(double[] weights, double[] moments){
		weights[5] = weights[5] + weights[4];
		moments[5] = moments[5] + moments[4];
	}
	private static void takeoffWeight(double[] weights, double[] moments){
		weights[5] = weights[5] - 7;
		moments[5] = moments[5] -336;
	}
	private static void printReport(double[] weights, double[] moments, double[] burn, String ident){
		double TOW, TOA, LDW, LDA;
		System.out.println("---------------------------------------------------------------------------------------------------------");
		System.out.println("Mount Royal Aviation Weight and Balance Report");
		System.out.print("Aircraft: Cessna 172");
		if(ident.equalsIgnoreCase("FSTP")){
			System.out.println("S C-" + ident.toUpperCase());
			System.out.println("Maximum Takeoff Weight: 2550lbs");
		}
		else{
			System.out.println("R C-" + ident.toUpperCase());
			System.out.println("Maximum Takeoff Weight: 2450lbs");
		}
		System.out.println("\n\t\t\tWeight\t\t\tArm\t\t\tMoment\n");
		System.out.println("Empty Weight:\t\t" + weights[5] + "\t\t\t" + Math.round((moments[5]/weights[5])*100.0)/100.0 + "\t\t\t" + moments[5]);
		System.out.println("Pilot & Front Seat: \t" + weights[0] + "\t\t\t37\t\t\t" + moments[0]);
		System.out.println("Rear Seat: \t\t" + weights[1] + "\t\t\t73\t\t\t" + moments[1]);
		System.out.println("Baggage - Area 1: \t" + weights[2] + "\t\t\t95\t\t\t" + moments[2]);
		System.out.println("Baggage - Area 2: \t" + weights[3] + "\t\t\t123\t\t\t" + moments[3]);
		System.out.println("------------------------------------------------------------------------------------");
		zeroFuel(weights, moments);
		System.out.println("Zero-Fuel Weight: \t" + Math.round((weights[5])*100.0)/100.0 + "\t\t\t" + Math.round((moments[5]/weights[5])*100.0)/100.0 + "\t\t\t" + moments[5]);
		System.out.println("Fuel (6lbs/US Gal): \t" + weights[4] + "\t\t\t48\t\t\t" + moments[4]);
		System.out.println("------------------------------------------------------------------------------------");
		rampWeight(weights, moments);
		System.out.println("Ramp Weight: \t\t" + Math.round((weights[5])*100.0)/100.0 + "\t\t\t" + Math.round((moments[5]/weights[5])*100.0)/100.0 + "\t\t\t" + moments[5]);
		System.out.println("Less Start & Taxi: \t-7\t\t\t48\t\t\t-336");
		System.out.println("------------------------------------------------------------------------------------");
		takeoffWeight(weights, moments);
		TOW = weights[5];
		TOA = Math.round((moments[5]/TOW)*100.0)/100.0;
		System.out.println("Take-Off Weight: \t" + Math.round((weights[5])*100.0)/100.0 + "\t\t\t" + Math.round((moments[5]/weights[5])*100.0)/100.0 + "\t\t\t" + moments[5]);
		System.out.println("Fuel Burn: \t\t" + Math.round((burn[0])*100.0)/100.0 + "\t\t\t48\t\t\t" + Math.round((burn[1])*100.0)/100.0);
		System.out.println("------------------------------------------------------------------------------------");
		landingWeight(weights, moments, burn);
		System.out.println("Landing Weight:  \t" + Math.round((weights[5])*100.0)/100.0 + "\t\t\t" + Math.round((moments[5]/weights[5])*100.0)/100.0 + "\t\t\t" + moments[5]);
		LDW = weights[5];
		LDA = Math.round((moments[5]/LDW)*100.0)/100.0;
		System.out.println("\nFirst Aid kit and Tow Bar (2lbs each) are considered permanent equipment at Front Seats and Baggage Area 2.\n");
		withinCofG(TOW, TOA, ident, LDA, burn, weights);
	}
	private static boolean withinCofG(double y, double x, String ident, double LDA, double[] burn, double[] weights)
	{
		String category = null;
		boolean safe = false;
		boolean valid = false;
		
		if ((weights[4] - Math.round(Math.abs(burn[0]))) < (burn[2])){ //checks if flight might go into reserve fuel. if landing fuel (T/O fuel - planned burn) < reserve, warn user. 
			System.out.println("WARNING: Flight may go into reserve fuel with these calculations. Please check and update values before flight.");
		}
		
		while(!valid){
			System.out.println("---------------------------------------------------------------------------------------------------------");
			System.out.print("Are you flying in the (U)tility or (N)ormal category? ");
			try{
				category = keyboard.next();
				if(category.equalsIgnoreCase("U") || category.equalsIgnoreCase("N")){
					valid = true;
				}
				else{
					System.out.println("Invalid Input. Please try again.\n");
					valid = false;
				}
			}
			catch(InputMismatchException e){
				System.out.println("Invalid selection. Please try again.\n");
			}
		}
		
		if(category.equalsIgnoreCase("U")){
			if (x >= 35 && x <= 40.5){		
					//checks full fore and full aft utility limits for both models
				if( y<= 2200 && y <= (100*x - 1550)){ 									//checks Max ramp weight and slope limits
					safe = true;
					System.out.println("\nC-" + ident.toUpperCase() +" is within CofG limits for this mission.");
					System.out.print("Have a good flight!");
				}
			}
			else{
				safe = false;
				System.out.println("\n!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!");
				System.out.println("WARNING: " + ident.toUpperCase() + " IS NOT WITHIN CENTER OF GRAVITY LIMITS");
				System.out.println("\tNOT WITHIN UTILITY CATEGORY LIMITS AT TAKEOFF");
				System.out.println("!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!\n");
				
				if(LDA >= 35 && LDA <= 40.5){
					System.out.println("Note: "+ ident.toUpperCase() +" should enter utility category before planned landing.");
				}			
				
				if(weights[1] > 5 || weights[2] > 5 || weights[3] > 5){ //checks if there is anything in the rear seat, or baggage compartments, excluding permanent items and A/C books
					if (weights[1] > 5){
						System.out.println("NOTE: Remove all non-permanent items from the Rear Seat area. It must be empty for Utility operations.");
					}
					if (weights[2] > 5){
						System.out.println("NOTE: Remove all non-permanent items from Baggage Area 1. It must be empty for Utility operations.");
					}
					if (weights[3] > 5){
						System.out.println("NOTE: Remove all non-permanent items from Baggage Area 2. It must be empty for Utility operations.");
					}
				}
			}
		}
		if(category.equalsIgnoreCase("N")){
			if (x >= 35 && x <= 47.3){							//checks full fore and full aft limits
				
				if(!ident.equalsIgnoreCase("FSTP") && y<= 2450 && y<= 100 * x - 1550){ 		//checks Max T/O weight and slope limits for R models
					safe = true;
					System.out.println("\nC-" + ident.toUpperCase() +" is within CofG limits for this mission.");
					System.out.println("Have a good flight!");
				}
				else if(ident.equalsIgnoreCase("FSTP") && y <= 2550 && y <= 100 * x - 1550){	//checks Max T/O weight and slope limits for S model
					safe = true;
					System.out.println("\nC-" + ident.toUpperCase() +" is within CofG limits for this mission.");
					System.out.println("Have a good flight!");
				}
				else{
					safe = false;
					System.out.println("\n!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!"); 	//reports outside normal category for both model
					System.out.println("WARNING: " + ident.toUpperCase() + " IS NOT WITHIN CENTER OF GRAVITY LIMITS");
					System.out.println("\tNOT WITHIN MAX TAKEOFF WEIGHT LIMITS");
					System.out.println("!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!");
				}
			}
			else{
				safe = false;															//reports outside normal category for both types
				System.out.println("\n!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!");
				System.out.println("WARNING: AIRCRAFT IS NOT WITHIN CENTER OF GRAVITY LIMITS");
				System.out.println("\tNOT WITHIN NORMAL CATEGORY LIMITS");
				System.out.println("!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!");
			}
		}
		return safe;
		}
	private static void promptEnd(){
		String response;
		boolean loop = true;
		while(loop){
			System.out.println("---------------------------------------------------------------------------------------------------------");
			System.out.print("Would you like to (E)xit or (R)estart? ");
			response = keyboard.next();
			if(response.equalsIgnoreCase("E")){
				System.out.println("Exiting...");
				System.exit(0);
			}
			else if(response.equalsIgnoreCase("R")){
				loop = false;
				System.out.println("\n---------------------------------------------------------------------------------------------------------\n");
				run();
			}
			else{
				System.out.println("Invalid input. Please try again.");
			}
		}
	}
	public static void main(String[] args) {
		run();
	}
}


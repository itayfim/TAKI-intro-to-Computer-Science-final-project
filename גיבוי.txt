/* Itay Fhima
   ID 312182009 */

   /* Final project in the Introduction to Computer Science course.
	  In this code there is a full impletetion of TAKI game according to the rules.
	  As part of the program, there is a use of pointers, dynamic allocations of arrays are performed,
	  and implementation of the "merge sort" algorithm with the efficiency of nlogn. */

#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <stdbool.h>
#include <string.h>
#include <stdlib.h>
#include <time.h>

//Define consts in the program
#define MAX_CARDS 50
#define MAX_NAME_LEN 21
#define FIRST_TURN_CARDS 4
#define CARDS_NUM_OPTIONS 9
#define CARDS_COLOR_OPTIONS 4
#define MAX_OPTIONS_FOR_CARDS 14
#define ONE 1
#define ZERO 0
#define TWO 2
#define THREE 3
#define FOUR 4
#define ZERO_CHAR '0'
const char* COLOR = "COLOR";
const char* TAKI = "TAKI";
const char* STOP = "STOP";
const char* CHANGE = "<->";
const char* PLUS = "+";

//Define the arrays for random function
const char* deck_cards_nums[] = { "1","2","3","4","5","6","7","8","9" }; //Deck nums options
const char cards_colors[] = { 'Y','R','B','G' }; //Colors options (for deck & player)
const char* cards_nums_and_operators[] = { "1","2","3","4","5","6","7","8","9", "+", "STOP", "<->", "TAKI", "COLOR" }; //Nums & operators options (for player)

//Define the structures in the program
typedef struct statistics { 
	int counter;
	char* nums_and_ops;
}STATISTICS;

typedef struct cards {
	char* nums_and_ops;
	char color;
}CARDS;

typedef struct player {
	CARDS* cards; //Dynamic cards array
	char name[MAX_NAME_LEN];
	int num_of_cards; //Logical cards array length
	int cardsARR_len; //Physical cards array length
}PLAYER;

//Anouncment of the functions in the program
void opening_message();
int get_num_of_players();
void get_name(PLAYER data[], int num_of_players);
void divide_first_turn(PLAYER* player);
void deck_new_card(CARDS* data);
void print_deck_card(CARDS* data);
void print_player_cards(PLAYER player);
void check_player_allocation(PLAYER* player);
void check_cards_allocation(CARDS* data, PLAYER* player);
bool winner(int num_of_cards);
int make_a_move(PLAYER* player, CARDS* deck, bool TAKIflag);
char pick_a_color(PLAYER player);
void take_a_new_card(PLAYER* player, STATISTICS* stats);
void operate_player_choice(PLAYER* player, CARDS* deck, int move, int* index, int* num_of_players, bool* STOPflag, bool* CHANGEflag, STATISTICS* stats);
CARDS* double_cards_array(CARDS* cards, int num_of_cards, PLAYER* player);
void deletion_a_card(PLAYER* player, int move);
void TAKI_move(PLAYER* player, CARDS* deck, int move, int* index, int* num_of_players, bool* STOPflag, bool* CHANGEflag, STATISTICS* stats);
void PLUS_move(PLAYER* player, CARDS* deck, int move, int* index, int* num_of_players, bool* STOPflag, bool* CHANGEflag, STATISTICS* stats);
void Index_handling(bool* STOPflag, int* PIndex, int* num_of_players, bool* CHANGEflag);
void reset_stats(STATISTICS* stats);
void sort_statistics(STATISTICS* stats, int size);
void merge(STATISTICS arr1[], int size1, STATISTICS arr2[], int size2, STATISTICS temp[]);
void copy_arr(STATISTICS* stats, STATISTICS temp[], int size);
void print_statistics(STATISTICS* stats);

void main() {
	srand(time(NULL)); //randomize the time start value
	opening_message(); //Printing the openings message
	int num_of_cards, num_of_players, move, PIndex = 0; //PIndex = Player Index
	bool STOPflag = false, CHANGEflag = false; //Flags for STOP / <-> cards
	char* name_of_winner = NULL; //String to the name of the winner
	CARDS deck; //The struct of the deck
	STATISTICS stats[MAX_OPTIONS_FOR_CARDS]; //An array for the statistics
	reset_stats(&stats); //Inserting 0's to the counters and placing the strings of the cards
	num_of_players = get_num_of_players(); //Iputing the num of the players in the game
	PLAYER* player = (int*)malloc(num_of_players * sizeof(PLAYER)); //Allocating the array of the players according to the num of the players
	check_player_allocation(player); //Checking the allocation
	get_name(player, num_of_players); //Inputing the name of each player
	for (int i = 0; i < num_of_players; i++) {
		player[i].cards = (int*)malloc(FIRST_TURN_CARDS * sizeof(CARDS)); //Allocating dynamic cards array for each player
		check_cards_allocation(player[i].cards, &player[i]); //Checking the allocation
		divide_first_turn(&player[i]); //Dividing 4 cards for each player for the first turn
	}
	num_of_cards = FIRST_TURN_CARDS; //defining 4 cards as the num of the cards in the first turn
	deck_new_card(&deck); //placing new card for the deck (just for the first turn)
	while (!winner(num_of_cards)) { //Cheking if num of cards = 0
			print_deck_card(&deck); //Printing the updated card in the deck
			print_player_cards(player[PIndex]); //Printing the cards of the current player
			move = make_a_move(&player[PIndex], &deck, false); //pick's up the choice of the player (take a new card or putting a card in the deck)
			if (move == ZERO) { //Checking if the player wants a new card fron the deck
				if (player[PIndex].num_of_cards == player[PIndex].cardsARR_len) //Checking if the logical size is equal to the physical size
					player[PIndex].cards = double_cards_array(player[PIndex].cards, player[PIndex].num_of_cards, player); //allocating new place in the array (double it for efficiency)
				take_a_new_card(&player[PIndex], &stats); //Taking a new card for the player
			}
			else { //if the player doesnt want a new card
				operate_player_choice(&player[PIndex], &deck, move, &PIndex, &num_of_players, &STOPflag, &CHANGEflag, stats); //playing the turn according to the player's card
			}
			num_of_cards = player[PIndex].num_of_cards; //Updating the num of the cards of the current player
			if (!num_of_cards) { //If the is a winner 
				name_of_winner = player[PIndex].name; //Taking the winner's name
				break; // Going out from the loop
			}
			Index_handling(&STOPflag, &PIndex, &num_of_players, &CHANGEflag); //Increasing or decreasing the index according to the conditions
	}
	printf("\nThe winner is... %s! Congratulations!\n\n", name_of_winner);
	print_statistics(stats); //Printing the statistic of the game
	for (int k = 0; k < num_of_players; k++) //Free the cards array for each player
		free(player[k].cards);
	free(player); //Free the players dynamic array
}

void opening_message() {
	//This function prints the opening message
	printf("************  Welcome to TAKI game !!! ***********\n\n");
}

int get_num_of_players() {
	//This function pick's up the num of the players
	int num_of_players;
	printf("Please enter the number of players:\n");
	scanf("%d", &num_of_players);
	if (num_of_players < 1) { //Input check - if the user inserted less than 1 player - printing an error end asking from the user to do it again
		printf("Invalid choice. please try again.\n");
		num_of_players = get_num_of_players();
	}
	return num_of_players;
}

void get_name(PLAYER data[], int num_of_players) {
	//This function pick's up the user's names
	for (int i = 0; i < num_of_players; i++) {
		printf("Please enter the first name of player #%d:\n", i+ONE);
		scanf("%s", data[i].name);
	}
}

void deck_new_card(CARDS* data) {
	//This function divides a new card in the start of the game to the deck 
	data->nums_and_ops = deck_cards_nums[rand() % CARDS_NUM_OPTIONS]; //Randomize 1-9 (without "special" cards)
	data->color = cards_colors[rand() % CARDS_COLOR_OPTIONS]; //Randomize the colors
}

void print_deck_card(CARDS* data) {
	//This function prints the deck card (upper card)
	printf("\nUpper card:\n");
	printf("*********\n");
	printf("*       *\n");
	if (data->nums_and_ops == COLOR)
		printf("* %s *\n", data->nums_and_ops);
	else if (data->nums_and_ops == TAKI || data->nums_and_ops == STOP)
		printf("*  %s *\n", data->nums_and_ops);
	else if (data->nums_and_ops == CHANGE)
		printf("*  %s  *\n", data->nums_and_ops);
	else 
		printf("*   %s   *\n", data->nums_and_ops);
	printf("*   %c   *\n", data->color);
	printf("*       *\n");
	printf("*********\n");
}

void divide_first_turn(PLAYER* player) {
	//This function divides four cards to each player in the start of the game
	player->cardsARR_len = FIRST_TURN_CARDS; 
	for (int i = 0; i < FIRST_TURN_CARDS; i++) {
		player->cards[i].nums_and_ops = cards_nums_and_operators[rand() % MAX_OPTIONS_FOR_CARDS]; //Randomize the cards 1-9 & special operators
		if (player->cards[i].nums_and_ops != COLOR) //no specific color for "COLOR" card
			player->cards[i].color = cards_colors[rand() % CARDS_COLOR_OPTIONS]; //Randomize the color (4 options)
	}
	player->num_of_cards = FIRST_TURN_CARDS;
}

void print_player_cards(PLAYER player) {
	//This function prints the player's cards
	printf("\n%s's turn:\n\n", player.name);
	for (int i = 0; i < player.num_of_cards; i++) {
		printf("\nCard #%d:\n", i + 1);
		printf("*********\n");
		printf("*       *\n");
		if (player.cards[i].nums_and_ops == COLOR)
			printf("* %s *\n", player.cards[i].nums_and_ops);
		else if (player.cards[i].nums_and_ops == TAKI || player.cards[i].nums_and_ops == STOP)
			printf("*  %s *\n", player.cards[i].nums_and_ops);
		else if (player.cards[i].nums_and_ops == CHANGE)
			printf("*  %s  *\n", player.cards[i].nums_and_ops);
		else
			printf("*   %s   *\n", player.cards[i].nums_and_ops);
		if (player.cards[i].nums_and_ops != COLOR) 
			printf("*   %c   *\n", player.cards[i].color);
		else
			printf("*       *\n");
		printf("*       *\n");
		printf("*********\n");
	}
}

bool winner(int num_of_cards) {
	//This function checks in every rutn if there is a winner 
	return (num_of_cards == ZERO);
}

void check_player_allocation(PLAYER* player) {
	//This function operated check for the dynamic allocation
	if (!player) {
		printf("ERROR! PLAYER memory allocation failed.\n");
		exit(1);
	}
}

void check_cards_allocation(CARDS* data, PLAYER* player) {
	//This function operated check for the dynamic allocation
	if (player->num_of_cards == ZERO)
		return;
	else if (!data) {
		printf("ERROR! CARDS memory allocation failed.\n");
		exit(1);
	}
}

int make_a_move(PLAYER* player, CARDS* deck, bool TAKIflag) {
	//This function pick's up the choice of the player (take a new card or putting a card in the deck)
	int move;
	if (TAKIflag) //In case of TAKI card
		printf("Please enter 0 if you want to finish your turn\n");
	else
		printf("Please enter 0 if you want to take a card from the deck\n");
	printf("or %d-%d if you want to put one of your cards in the middle:\n", ONE, player->num_of_cards);
	scanf("%d", &move);
	if (player->cards[move - ONE].nums_and_ops != COLOR && move != ZERO) { 
		if (player->cards[move - ONE].color != deck->color || move > player->num_of_cards) { //Input check
			printf("Invalid card! Try again.\n");
			move = make_a_move(player, deck, TAKIflag);
		}
	}
	return move;
}

char pick_a_color(PLAYER player) {
	//This function pick's up the player color choice in case of COLOR card
	char ch;
	int choice;
	printf("Please enter your color choice:\n");
	printf("%d - Yellow\n", ONE);
	printf("%d - Red\n", TWO);
	printf("%d - Blue\n", THREE);
	printf("%d - Green\n", FOUR);
	scanf("%d", &choice);
	switch (choice)
	{
	case ONE:
		ch = cards_colors[ZERO];
		break;
	case TWO:
		ch = cards_colors[ONE];
		break;
	case THREE:
		ch = cards_colors[TWO];
		break;
	case FOUR:
		ch = cards_colors[THREE];
		break;
	default: //Input check
		printf("Invalid input. please Try again.\n");
		ch = pick_a_color(player);
		break;
	}
	return ch;
}

CARDS* double_cards_array(CARDS* cards, int num_of_cards, PLAYER* player) {
	//This function doubles the cards of the array in case that the pkayer needs a new card and the array is isn't big enough
	player->cardsARR_len = num_of_cards * TWO; //double the physical length
	CARDS* temp = (CARDS*)malloc(player->cardsARR_len * sizeof(CARDS)); //allocating new temporary array
	check_cards_allocation(temp, player);
	for (int i = 0; i < num_of_cards; i++) { //copying the values to the new array
		temp[i].color = cards[i].color;
		temp[i].nums_and_ops = cards[i].nums_and_ops;
	}
	free(cards);
	cards = temp;
	return cards; //returning the address of the new doubled array
}

void take_a_new_card(PLAYER* player, STATISTICS* stats) {
	//This function randomly selects a number and color, and also updates the statistics array
	int RNDindex;
	RNDindex = rand() % MAX_OPTIONS_FOR_CARDS; //for the nums and operators - 14 different options
	player->cards[player->num_of_cards].nums_and_ops = cards_nums_and_operators[RNDindex];
	if (player->cards[player->num_of_cards].nums_and_ops != COLOR) //check if the cards isn't COLOR
		player->cards[player->num_of_cards].color = cards_colors[rand() % CARDS_COLOR_OPTIONS];
	player->num_of_cards += ONE; //updating the cards num of the player
	stats[RNDindex].counter++; //increasing the counter of the relevant num/operator
}

void operate_player_choice(PLAYER* player, CARDS* deck, int move, int* index, int* num_of_players, bool* STOPflag, bool* CHANGEflag, STATISTICS* stats) {
	//This function operates the player choice, That is, the function executes the card (if it is a special card)
	deck->nums_and_ops = player->cards[move - ONE].nums_and_ops; //updating the deck
	if (player->cards[move - ONE].nums_and_ops == COLOR) { //In case of COLOR
		deck->color = pick_a_color(*player);
		deletion_a_card(player, move);
	}
	else {
		deck->color = player->cards[move - ONE].color; //updating the deck color
		deletion_a_card(player, move);
		if (deck->nums_and_ops == TAKI) //In case of TAKI
			TAKI_move(player, deck, move, index, num_of_players, STOPflag, CHANGEflag, stats);
		else if (deck->nums_and_ops == STOP) { //in case of STOP
			*STOPflag = true;
			if (*num_of_players == 2 && player->num_of_cards == 0) { //if the last card that placed is STOP
				printf("Placing a %s as the last card, according to the rules you must take another card.\n", STOP);
				take_a_new_card(player, stats);
			}
		}
		else if (deck->nums_and_ops == CHANGE) { //in case of <->
			if (*CHANGEflag) //if the flag is on - turn it off
				*CHANGEflag = false;
			else //if the flag is off - turn it on
				*CHANGEflag = true;
		}
		else if (deck->nums_and_ops == PLUS) //in case of +
			PLUS_move(player, deck, move, index, num_of_players, STOPflag, CHANGEflag, stats);	
	}
}

void deletion_a_card(PLAYER* player, int move) {
	//This functions is replacing the card that placed in the last card
	player->cards[move - ONE].color = player->cards[player->num_of_cards - ONE].color;
	player->cards[move - ONE].nums_and_ops = player->cards[player->num_of_cards - ONE].nums_and_ops;
	player->num_of_cards -= ONE; //updating the cards num of the player
}

void TAKI_move(PLAYER* player, CARDS* deck, int move, int* index, int* num_of_players, bool* STOPflag, bool* CHANGEflag, STATISTICS* stats) {
	//This function operates the TAKI card moves
	while (player->num_of_cards != ZERO) { //in case of "end game" - get out from the loop
		print_deck_card(deck);
		print_player_cards(*player);
		move = make_a_move(player, deck, true); //inputing the player's move
		if (move == ZERO) //get out from the loop if the player wants to stop the cards placing
			break;
		deck->nums_and_ops = player->cards[move - ONE].nums_and_ops;
		if (player->cards[move - ONE].nums_and_ops == COLOR) { //In case of COLOR
			deck->color = pick_a_color(*player);
			deletion_a_card(player, move);
			break; //According to the orders - if the player placed COLOR card - exit the loop
		}
		else {
			deck->color = player->cards[move - ONE].color;
			deletion_a_card(player, move);
		}
	}
	//Performs all necessary checks at the end of the loop
	if (deck->nums_and_ops == STOP) { //in case of STOP
		*STOPflag = true;
		if (*num_of_players == 2 && player->num_of_cards == 0) {
			printf("Placing a %s as the last card, according to the rules you must take another card.\n", STOP);
			take_a_new_card(player, stats);
		}
	}
	else if (deck->nums_and_ops == CHANGE) { //in case of <->
		if (*CHANGEflag)
			*CHANGEflag = false;
		else
			*CHANGEflag = true;
	}
	else if (deck->nums_and_ops == PLUS) //in case of +
		PLUS_move(player, deck, move, index, num_of_players, STOPflag, CHANGEflag, stats);
}

void PLUS_move(PLAYER* player, CARDS* deck, int move, int* index, int* num_of_players, bool* STOPflag, bool* CHANGEflag, STATISTICS* stats) {
	////This function operates the PLUS card moves
	print_deck_card(deck);
	print_player_cards(*player);
	move = make_a_move(player, deck, *STOPflag);
	if (move == ZERO) { //in case that the player wants a new card from the deck
		if (player->num_of_cards == player->cardsARR_len) //Checking if the logical size is equal to the physical size
			player->cards = double_cards_array(player->cards, *num_of_players, player); //allocating new place in the array (double it for efficiency)
		take_a_new_card(player, stats);
		return;
	}
	if (player->num_of_cards != ZERO)
		operate_player_choice(player, deck, move, index, num_of_players, STOPflag, CHANGEflag, stats);
	else {
		printf("Placing a %s as the last card, according to the rules you must take another card.\n", PLUS);
		take_a_new_card(player, stats);
	}
}

void Index_handling(bool* STOPflag, int* PIndex, int* num_of_players, bool* CHANGEflag) {
	// This function performs calculations for the index in the main loop by the flags
	if (*STOPflag && *CHANGEflag == false) {
		if (*PIndex + TWO == *num_of_players)
			*PIndex = ZERO;
		else
			*PIndex = *PIndex + TWO;
		*STOPflag = false;
		return;
	}
	else if (*STOPflag && *CHANGEflag == true) {
		if (*PIndex - TWO == ZERO)
			*PIndex = *num_of_players - ONE;
		else
			*PIndex = *PIndex - TWO;
		*STOPflag = false;
		return;
	}
	if (*CHANGEflag) {
		if (*PIndex - ONE == -ONE)
			*PIndex = *num_of_players - ONE;
		else
			*PIndex -= ONE;
	}
	else {
		if (*PIndex + ONE == *num_of_players)
			*PIndex = ZERO;
		else
			*PIndex += ONE;
	}
}

void reset_stats(STATISTICS* stats) {
	//This function resets the statistics array. Enter 0 anywhere in the counter, and enter the card names in the appropriate place
	for (int i = 0; i < MAX_OPTIONS_FOR_CARDS; i++) {
		stats[i].nums_and_ops = cards_nums_and_operators[i]; //using the const array 
		stats[i].counter = ZERO;
	}
}

void sort_statistics(STATISTICS* stats, int size) {
	// This function sorts the statistics array by the technic of "Merge Sort".
	// O(n) = n*log(n)
	if (size <= ONE) //exit condition
		return;
	// divide the array into 2 parts
	sort_statistics(stats, size / TWO);
	sort_statistics(stats + size / TWO, size - size / TWO);
	STATISTICS* temp = (STATISTICS*)malloc(size * sizeof(STATISTICS)); //temporary dynamic array
	if (temp) {
		merge(stats, size / TWO, stats + size / TWO, size - size / TWO, temp); //mergeing the 2 parts in a sorted manner
		copy_arr(stats, temp, size);
		free(temp);
	}
	else { 
		printf("ERROR! STATISTICS memory allocation failed.\n");
		exit(1);
	}
}

void merge(STATISTICS arr1[], int size1, STATISTICS arr2[], int size2, STATISTICS temp[]) {
	//This function merges in a sorted manner all the arrays sent recursively from the main sort function
	int index1 = ZERO, index2 = ZERO, tempIndex = ZERO;
	while (index1 < size1 && index2 < size2) {
		if (arr1[index1].counter >= arr2[index2].counter) {
			temp[tempIndex].counter = arr1[index1].counter;
			temp[tempIndex].nums_and_ops = arr1[index1].nums_and_ops;
			index1++;
		}
		else {
			temp[tempIndex].counter = arr2[index2].counter;
			temp[tempIndex].nums_and_ops = arr2[index2].nums_and_ops;
			index2++;
		}
		tempIndex++;
	}
	while (index1 < size1 || index2 < size2) {
		if (index1 == size1) {
			temp[tempIndex].counter = arr2[index2].counter;
			temp[tempIndex].nums_and_ops = arr2[index2].nums_and_ops;
			index2++;
		}
		else if (index2 == size2) {
			temp[tempIndex].counter = arr1[index1].counter;
			temp[tempIndex].nums_and_ops = arr1[index1].nums_and_ops;
			index1++;
		}
		tempIndex++;
	}
}

void copy_arr(STATISTICS* stats, STATISTICS temp[], int size) {
	//This function copying the values from the temporary array to the oroginal array
	//stats - dest, temp - source
	for (int i = 0; i < size; i++) {
		stats[i].counter = temp[i].counter;
		stats[i].nums_and_ops = temp[i].nums_and_ops;
	}
}

void print_statistics(STATISTICS* stats) {
	//This function prints the statistics in the end of the game
	sort_statistics(stats, MAX_OPTIONS_FOR_CARDS);
	printf("************ Game Statistics ************\n");
	printf("Card # | Frequency\n");
	printf("__________________\n");
	for (int i = 0; i < MAX_OPTIONS_FOR_CARDS; i++) {
		if (stats[i].nums_and_ops == COLOR)
			printf(" %s |    %d\n", stats[i].nums_and_ops, stats[i].counter);
		else if (stats[i].nums_and_ops == TAKI || stats[i].nums_and_ops == STOP)
			printf(" %s  |    %d\n", stats[i].nums_and_ops, stats[i].counter);
		else if (stats[i].nums_and_ops == CHANGE)
			printf("  %s  |    %d\n", stats[i].nums_and_ops, stats[i].counter);
		else
			printf("   %s   |    %d\n", stats[i].nums_and_ops, stats[i].counter);
	}
}






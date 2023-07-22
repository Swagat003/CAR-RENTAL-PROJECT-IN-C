#include <stdio.h>		// standard input output library
#include <string.h>		// string library functions to work with strings
#include <time.h>		// library functions for manipulating date and time

#define MAX_CAR 10

// car structure for storing cars details
typedef struct car{
	char brand[50];
	char model[50];
	int year;
	int rate;
	int available;
}car;

// profile structure for storing user details
typedef struct profile
{
	char name[20];
	int id;
	int pasw;
}profile;

// rentData structure for storing rent details
typedef struct rentData
{
	int u_id;
	int carNo;
	int dr,mr,yr; //rent date
	int dret,mret,yret; //return date
	int price;
}rentData;

// cars list
car cars[MAX_CAR]={
	{"HYUNDAI","Veloster",2020,200,1},
	{"SUZUKI","Celerio",2022,250,1},
	{"HONDA","Civic",2019,300,1},
	{"TOYOTA","Avalon",2021,200,1},
	{"FORD","Explorer",2022,250,1},
	{"TATA","Safari",2022,200,1},
	{"AUDI","Q3",2021,250,1}
};

profile user;	// global profile structure type variable declaration
rentData rent;	// global rentData structure type variable declaration

// function for display car list in output
void display(){
	int i,check=0;
	printf("+------+-----------+-------------+----------+----------------+\n");
	printf("|CarNo.| %-10s|   %-10s|   %-7s| %-15s|\n","BRAND","MODEL","YEAR","RENT PER DAY");
	printf("+------+-----------+-------------+----------+----------------+\n");
	for(i=0;i<MAX_CAR;i++){
		if(cars[i].available==1){
			printf("|  %2d  | %-10s|   %-10s|   %-7d|   %-13d|\n",i+1,cars[i].brand,cars[i].model,cars[i].year,cars[i].rate);
			check=1;
		}
	}
	if(check==0){
		printf("Sorry! No car is available\n");
	}
	printf("+------+-----------+-------------+----------+----------------+\n");
	return;
}

// function for login for existing user account
int login(){
	int id;
	int pasw;
	profile check;
	printf("Enter ID: ");
	scanf("%d",&id);
	printf("Enter password: ");
	scanf("%d",&pasw);
	FILE *f1;
	f1=fopen("User_data.dat","r");
	while (fread(&check,sizeof(profile),1,f1))
	{
		if(check.id==id&&check.pasw==pasw){
			user=check;
			fclose(f1);
			return 1;
		}
	}
	fclose(f1);
	return 0;
}

// function for creating new account and storing user details in file
void signup(){
	FILE *f1,*f2;
	f1 = fopen("User_data.dat","a");
	f2 = fopen("User_data.dat","r");
	profile input,check;
	fflush(stdin);
	printf("Name: ");
	scanf("%[^\n]s",input.name);
	printf("ID: ");
	scanf("%d",&input.id);
	while (fread(&check,sizeof(profile),1,f2))
	{
		if(check.id==input.id){
			printf("ID already exist\n\n");
			fclose(f1);
			fclose(f2);
			chooesLogin();
			return;
		}
	}
	printf("Create password: ");
	scanf("%d",&input.pasw);
	fwrite(&input,sizeof(profile),1,f1);
	printf("Sign up succesfuly\n\n");
	printf("Login\n");
	fclose(f1);
	fclose(f2);

	return;
}

// function for selecting login or signup
void chooesLogin(){
	int login_method;
	while (1)
	{
		printf("Chooes your login method:\n");
		printf("1.Login\n2.Sign up\n");
		scanf("%d",&login_method);
		if(login_method==1){
			label:
			if(login())
			{
			printf("\n--------------------------------------------------------------------------------\n");
			printf("Login succesfuly\n");
			printf("Name: %s\nID: %d\n",strupr(user.name),user.id);
			return;
			}
			else
			{
				printf("Wrong password or ID\n\n");
				chooesLogin();
				return;
			}
		}else if(login_method==2){
			signup();
			goto label;
			return;
		}else{
			printf("\nWrong choice!! Try again.\n");
		}
	}
	return;
}

// function for count total number of days between two dates
int daysCount(){
	int days=0;
	
	// Calculate time in seconds for each date
    struct tm date1 = { .tm_mday = rent.dr, .tm_mon = rent.mr - 1, .tm_year = rent.yr - 1900 };
    struct tm date2 = { .tm_mday = rent.dret, .tm_mon = rent.mret - 1, .tm_year = rent.yret - 1900 };
    time_t t1 = mktime(&date1);
    time_t t2 = mktime(&date2);

    // Calculate difference in seconds
    double diff = difftime(t2, t1);

    // Convert difference to number of days
    days = (int) (diff / (24 * 60 * 60));

	return days;
}

// function for deleteing a rentData structure record from file
void deleteRecord(int inp_carNo){
	rentData temp;
	FILE *f1,*f2;
	f1=fopen("Rented_car_data.dat","r");
	f2=fopen("temp.dat","w");
	fclose(f2);
	f2=fopen("temp.dat","a");
	while (fread(&temp,sizeof(rentData),1,f1))
	{
		if(temp.carNo!=inp_carNo || temp.u_id!=user.id){
			fwrite(&temp,sizeof(rentData),1,f2);
		}
	}
	fclose(f1);
	fclose(f2);
	
	f1=fopen("Rented_car_data.dat","w");
	fclose(f1);
	f1=fopen("Rented_car_data.dat","a");
	f2=fopen("temp.dat","r");
	while (fread(&temp,sizeof(rentData),1,f2))
	{
		fwrite(&temp,sizeof(rentData),1,f1);
	}
	fclose(f2);
	fclose(f1);
	f2=fopen("temp.dat","w");
	fclose(f2);
	return;
}

// funtion for rent a car
void car_rent(){
	int sl;
	printf("\nWELCOME DEAR CUSTOMER!!! \n");
	display();
	printf("ENTER THE CarNO. OF THE CAR YOU WANT TO RENT: ");
	scanf("%d",&sl);
	sl--;
	printf("BRAND: %s\nMODEL: %s\nYEAR: %d\nRENT PER DAY: %d/-\n\n",cars[sl].brand,cars[sl].model,cars[sl].year,cars[sl].rate);
	printf("ENTER DATE ON WHICH YOU WILL TAKE THE CAR(dd mm yyyy): ");
    scanf("%d%d%d",&rent.dr,&rent.mr,&rent.yr);
    printf("ENTER THE DATE ON WHICH YOU WILL RETURN THE CAR(dd mm yyyy): ");
    scanf("%d%d%d",&rent.dret,&rent.mret,&rent.yret);
	rent.u_id=user.id;
	rent.carNo=sl;
	int days = daysCount();
	rent.price=cars[sl].rate*days;
	FILE *f1;
	f1 = fopen("Rented_car_data.dat","a");
	fwrite(&rent,sizeof(rentData),1,f1);
	fclose(f1);
	printf("Car rented sucessfuly\n\n");
	printf("Details:\n");
	printf("-----------------------------------------------------------------------------------------------------------\n");
	printf("NAME: %s\nCUSTOMER ID: %d\nCAR RENTED: %s %s\nNUMBER OF DAYS: %d\nRENT: %d/-\n",strupr(user.name),user.id,cars[sl].brand,cars[sl].model,days,cars[sl].rate*days);
	printf("-----------------------------------------------------------------------------------------------------------\n");
	printf("WARNING: If any damage is done to the car then you are entirely responsible. The car has to be returned in its initial condition.\n");
	printf("-----------------------------------------------------------------------------------------------------------\n");
	FILE *f2;
	char fileName[50];
	strcat(strcpy(fileName,user.name),"_bill.txt");
	f2=fopen(fileName,"w");
	fprintf(f2,"Details:\n");
	fprintf(f2,"-----------------------------------------------------------------------------------------------------------\n");
	fprintf(f2,"NAME: %s\nCUSTOMER ID: %d\nCAR RENTED: %s %s\nNUMBER OF DAYS: %d\nRENT: %d/-\n",strupr(user.name),user.id,cars[sl].brand,cars[sl].model,days,cars[sl].rate*days);
	fprintf(f2,"-----------------------------------------------------------------------------------------------------------\n");
	fprintf(f2,"WARNING: If any damage is done to the car then you are entirely responsible. The car has to be returned in its initial condition.\n");
	fprintf(f2,"-----------------------------------------------------------------------------------------------------------\n");
	fclose(f2);
	return;
}

// function for return a car
void car_return(){
	printf("\nWelcome back dear customer \n");
	FILE *f1;
	f1 = fopen("Rented_car_data.dat","r");
	rentData check;
	int flag=0;
	while (fread(&check,sizeof(rentData),1,f1))
	{
		if(check.u_id == user.id)
		{
			if(flag==0){
			printf("Rented cars of yours\n");
			}
			rent=check;
			printf("\n%s %s\nCarno.: %d\nRent date:%02d-%02d-%04d\nReturn date:%02d-%02d-%04d\n",cars[rent.carNo].brand,cars[rent.carNo].model,rent.carNo,check.dr,check.mr,check.yr,check.dret,check.mret,check.yret);
			flag=1;
		}
	}
	printf("\nEnter carno. which you want to return\n");
	int in_car;
	scanf("%d",&in_car);
	deleteRecord(in_car);
	if(flag==0){
		printf("no car found to reaturn\n");
	}
	printf("thank you!!");
	return;
}

int main(){
	printf("\tWELCOME TO OUR CAR RENTAL SYSTEM!!\n");
	chooesLogin();
	int n1;
	label:
		printf("\n\aHI!! ARE YOU HERE TO TAKE THE CAR SERVICE OR RETURN BACK THE CAR? \n 1.WANT TO RENT A CAR.\n 2.WANT TO RETURN THE CAR.\n");
		scanf("%d",&n1);
		switch(n1)
    	{
    	    case 1:{car_rent();
        	break;}
        	case 2:car_return();
        	break;
        	default:{printf("Wrong choice!! Try again.\n");
        	goto label;}
    	}
	return 0;
}

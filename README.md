# 11613061_OS-project1
Question 29

#include<stdio.h>
#include<stdlib.h>
const int instance=4;
void safeSequence(int process[],int avail[],int max[][instance], int allocation[][instance],int need[][instance])
{
	void step2(int);
	int sequence[5];
	int s=0;
	bool safe;
	int work[5];
	for(int i=0;i<4;i++)
	{
		work[i]=avail[i];
	}
	bool finish[]={false,false,false,false,false};
	int terminate=0;
	while(terminate<5)
	{
		for(int i=0;i<5;i++)
		{
			if(finish[i]==false)
			{
				int k;
				for(k=0;k<4;k++)
				{
					if(need[i][k]>work[k])
					{
						break;
					}
				}
				if(k==instance)
				{
					for(int j=0;j<4;j++)
					{
						work[j]+=allocation[i][j];
					}
					finish[i]=true;
					sequence[s++]=i;
					terminate++;
				}
			}
		}
	}
	for(int i=0;i<5;i++)
	{
		if(finish[i]==true)
		safe=true;
		else
		safe=false;
	}
	if(safe=true)
	{
		printf("\nSafe Sequence is : ");
		for(int i=0;i<5;i++)
		{
			printf("P%d  ",sequence[i]);
		}
		printf("\n");
	}
	else
	{
		printf("\nSystem is not in safe Sequence");
	}
}
int main()
{
	int process[]={0,1,2,3,4};
	int avail[]={1,5,2,0};
	int allocation[][4]={{0,0,1,2},
						 {1,0,0,0},
						 {1,3,5,4},
						 {0,6,3,2},
						 {0,0,1,4}
						};
	int max[][4]= {{0,0,1,2},
				{1,7,5,0},
		 		{2,3,5,6},
				{0,6,5,2},
				{0,6,5,6}
	 			};
	int need[5][4];
	for(int i=0;i<5;i++)
	{
		for(int j=0;j<4;j++)
		{
			need[i][j]=max[i][j]-allocation[i][j];	
		}
	}
	printf("\nProcess\t\tMaximum\t\tAllocation\tNeed");
	printf("\n\t\tA B C D\t\tA B C D\t\tA B C D\n");
	for(int i=0;i<5;i++)
	{
		printf("P%d",i);
		printf("\t\t%d %d %d %d",max[i][0],max[i][1],max[i][2],max[i][3]);
		printf("\t\t%d %d %d %d",allocation[i][0],allocation[i][1],allocation[i][2],allocation[i][3]);
		printf("\t\t%d %d %d %d\n",need[i][0],need[i][1],need[i][2],need[i][3]);
	}
	printf("\nAvailable are A B C D\n");
	printf("              ");	
	for(int i=0;i<4;i++)
	{
		printf("%d ",avail[i]);
	}
	printf("\n");
	safeSequence(process,avail,max,allocation,need);
}


Question 4

#include<stdio.h>
#include<unistd.h>
#include<semaphore.h>
#include<stdlib.h>
#include<malloc.h>
#include<pthread.h>
#include<time.h>
struct Process {
	int time,At,Bt,id;
	sem_t se;
	clock_t arrival;
	int flag,completed,p;
	struct Process *next;
};
int i=0,k=0;
typedef struct Process node;
float TAT=0,WT=0;
clock_t st;
node *sp1=NULL,*sp2=NULL,*temp;
void sort_arrival() {
	temp=(node *)malloc(sizeof(node));
	printf("\nEnter Arrival Time of %d Process :",(i+1));
	scanf("%d",&temp->At);
	printf("Enter Burst Time :");
	scanf("%d",&temp->time);
	temp->id=i+1;
	temp->p=1;
	temp->flag=1;
	temp->Bt=temp->time;
	sem_init(&temp->se,0,0);
	int t=temp->At;
	node *st=sp1;
	if (st->At > t) {
        	temp->next = sp1;
        	sp1=temp;
    	}
    	else {
        	while (st->next != NULL && st->next->At < t) {
            		st = st->next;
        	}
        temp->next = st->next;
        st->next = temp;
    }
}
void *processor(node *S) {
	
	clock_t count;
	while(1) {
		sem_wait(&S->se);
		if((S->At<=(clock()-st)/CLOCKS_PER_SEC && S->p==1)) {
			S->p=0;
			count=clock();
		}
		if(S->flag==1) {
			printf("\nProcess-%d Running \nTimer :%d",S->id,(clock()-st)/CLOCKS_PER_SEC);
			S->flag=0;
			S->arrival=clock();
		}
		if((clock()-count)/CLOCKS_PER_SEC==1) {
			count=clock();
			printf("\nTimer :%d",(clock()-st)/CLOCKS_PER_SEC);
			S->time-=1;
			if(S->time==0) {
				TAT+=(clock()-st)/CLOCKS_PER_SEC-S->At;
				WT+=((clock()-st)/CLOCKS_PER_SEC)-S->Bt-S->At;
				sleep(2);
				node *st=sp2;
				while(st!=NULL) {
					if(st->next==S) {
						st->next=S->next;
						break;
					}
					if(sp2==S) {
						sp2=sp2->next;
						break;
					}
					st=st->next;
				}
				printf("\nProcess-%d Completed ",S->id);
				if(sp2!=NULL){
					printf("next Process-%d",sp2->id);
				}
				S->completed=7;
				if(sp2!=NULL){
					sem_post(&sp2->se);
				}
			}
			}
					if(S->completed==7) {
			break;
		}
	sem_post(&S->se);
	}
} 
void sort_burst(node *temp) {	//sort by burst time
	node *st=sp2;
	if(sp2==NULL) {
		sp2=temp;
		sp2->next=NULL;
	}
	else{
	int t=temp->time;
	if (st->time > t) {
        	temp->next = sp2;
        	sp2=temp;
    	}
    	else {
        	while (st->next != NULL && st->next->time< t) {
            		st = st->next;
        	}
        temp->next = st->next;
        st->next = temp;
    }
	}
}

 void main() {
	printf("\t\t\t\t\t\t\t\t\t*********Shortest Job First**********\n");
	int n,l=1;
	pthread_t p[10];
	printf("\nEnter No.of Processes :");
	scanf("%d",&n);
	while(i<n) {
		if(sp1==NULL) {
			sp1=(node *)malloc(sizeof(node));
			printf("Enter Arrival Time of %d Process :",(i+1));
			scanf("%d",&sp1->At);
			printf("Enter Burst Time :");
			scanf("%d",&sp1->time);
			sp1->id=i+1;
			sp1->flag=1;
			sp1->p=1;
			sp1->Bt=sp1->time;
			sem_init(&sp1->se,0,0);
			sp1->next=NULL;
		}
		else {
			sort_arrival();
		}
		i++;
	}
	i=0;
	system("cls");
	printf("\t\t\t\t\t\t\t\t**********PROCESSOR************\n\n");
	st=clock();
	while(i<n) {
		temp=sp1;	
		if(temp->At<=0) {
			printf("Process-%d is Dicarded Due to Incorrect Arrival Time\n",temp->id);
			sp1=temp->next;
			temp=sp1;
			i++;
		}
		if(l==1) {
				l=0;
				sem_post(&temp->se);
			}
		if((clock()-st)/CLOCKS_PER_SEC==temp->At) {
			printf("Process-%d is created\n",temp->id);
			pthread_create(&p[i],NULL,processor,temp);
			sp1=sp1->next;
			sort_burst(temp);
			i++;
		}
	}
	for(i=0;i<n;i++) {
		pthread_join(p[i],NULL);
	}
	printf("\nAverage Waiting Time :%f\nAverage Turn Around Time :%f",(float)WT/n,(float)TAT/n);
}

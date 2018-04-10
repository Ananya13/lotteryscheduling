# lotteryscheduling
#include<stdio.h>
#include<pthread.h>
#include<semaphore.h>
struct lot
{
    int arv_time
    int burst_time;
    int lottery_val;

}s[10];

int ticket[10][10],lottery[10];
int winner;
int p = 0;
int size=10;
int quantom=20;

sem_t common;

//assuming all process at same time
void rand_gen_arv_time(int size,int limit)
{
    for(int i=0;i<size;i++)
    {
        /*int temp=(rand()%limit)+1;
        s[i].arv_time=temp;*/

        s[i].arv_time=0;
    }
}

void rand_gen_burst_time(int size,int limit)
{
    for(int i=0;i<size;i++)
    {
        int temp=(rand()%limit)+1;
        s[i].burst_time=temp;
    }
}

void lott()
{

    p=0;
    for (int i = 0; i < size; i++) 
    {
        if ((s[i].burst_time > 0)) 
        {
            lottery[i] = s[i].burst_time / 20;
            if ((lottery[i] == 0) && (s[i].burst_time > 0))
                lottery[i] = 1;
        }       
        else
            lottery[i]=0;

        for (int z = 0; z < lottery[i]; z++) 
        {
            ticket[i][z] = p++;
        }

    }

            
}

int sum_req()
{
    int sum=0;
    for(int i=0;i<size;i++)
        sum+=s[i].burst_time;
    return sum;
}

void remove_node(struct lot *temp)
{
    int i=0;
    while(&s[i]!=temp)
        i++;

    if(i!=size)
        while(i<size-1)
        {   s[i].arv_time=s[i+1].arv_time;
            s[i].burst_time=s[i+1].burst_time;
            s[i].lottery_val=s[i+1].lottery_val;
            i++;
        }
    else
        size--;
}

void *pro(int * loc)
{
    int temp1=*loc;
    printf("Process %d is being executed \n",temp1+1);
    sem_wait(&common);
    printf("entering critical section of %d \n",temp1+1);
    
    printf(" %d \t\t %d \t\t %d \n",temp1+1,s[temp1].arv_time,s[temp1].burst_time,lottery[temp1]);

    if ((s[temp1].burst_time > 0))  
    {
        s[temp1].burst_time -= quantom;
        lottery[temp1]=0;
        if (s[temp1].burst_time < 0) 
        {
            s[temp1].burst_time=0;
            printf(" %d \t\t %d \t\t %d \n",temp1+1,s[temp1].arv_time,s[temp1].burst_time,lottery[temp1]);
            remove_node(&s[temp1]);
            size--;
        }
        else
            printf(" %d \t\t %d \t\t %d \n",temp1+1,s[temp1].arv_time,s[temp1].burst_time,lottery[temp1]);

        printf("exiting critical section of %d \n",temp1+1);
        sem_post(&common);
        pthread_exit(NULL);
    }

}

int main()
{
    pthread_t v[10];
    sem_init(&common,0,1);
    srand(getpid());

    rand_gen_arv_time(10,30);
    rand_gen_burst_time(10,200);
    while(sum_req())
    {
        lott();
        printf("Process_No\tArv_time\tBrst_time\tNo_lottery\tLottery_tickets\n");
        printf("-----------------------------------------------------------------------------------------------------------------------\n");
        for(int i=0;i<size;i++)
        {
            printf(" %d \t\t %d \t\t %d \t\t %d \t\t %d to %d\n",i+1,s[i].arv_time,s[i].burst_time,lottery[i],ticket[i][0],ticket[i][0]+lottery[i]);
        }


        winner=rand()%p;

        printf("\nLucky winner is process no. %d\n\n",winner+1); //winner

        int temp1;
        for(int i =0;i<size;i++)
            for(int z=0;z<lottery[i];z++)
                if(ticket[i][z]==winner)
                    temp1=i;



        pthread_create(&v[temp1],NULL,pro,(void*) &temp1);
        pthread_join(v[temp1],NULL);




    }
    printf("------------------------------------------------------------------------------------------------------\nterminating \n");

}

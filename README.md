# CPUDiskSimulator
//Runs 10,000 processes through the CPU if the process is complete it will exit the program, if it is not it will go another round through the Disk drive and then again through the CPU until 10,000 runs.
#include <iostream>
#include <cmath>
#include <cstdlib>
#include <ctime>
#include <queue> 
#include <fstream>

using namespace std;

struct Process {
    int ID;
    double arrivalTime;
    double completionTime;
    bool complete;

    Process() {
        this->complete = false;
        this->arrivalTime = -1;
    }
};

struct Node {
    Process process;
    Node* next;
};

double cTime = 0;
int done = 0;
int MAX_PROCESSES = 10000;
queue<Process> rQCPU;
queue<Process> rQDisk;
bool cpuState=false;
bool diskState=false;

double CPU(double cpuSTime, double diskSTime, Process& p);
void readyQCPU(double cpuSTime, double diskSTime, Process& p);
void disk(double cpuSTime, double diskSTime, Process& p);
void readyQDisk(double cpuSTime, double diskSTime, Process& p);
double avgTurnAroundTime(Process process[], int MAX_PROCESSES);
double avgProcInDisk(double lambda);
double avgProcInCPU(double lambda);
int terminateCount();

int main() 
{
    srand(time(NULL));
    double cpuSTime = 0;
    double diskSTime = 0;
    double lambda = 0;
    Process process[MAX_PROCESSES];

    cout << "Input Lambda: ";
    cin >> lambda;
    cout << "What is the average CPU service time: ";
    cin >> cpuSTime;
    cout << "What is the average Disk Service time: ";
    cin >> diskSTime;

    for (int i = 0; i < MAX_PROCESSES; i++) 
    {
        process[i].ID = i;
    }

    // The do-while loop will keep executing until all processes are completed
    do {
        for (int i = 0; i < MAX_PROCESSES; i++) 
        {
            if (!process[i].complete) 
            {
                readyQCPU(cpuSTime, diskSTime, process[i]);
            }
        }
    } while (done != MAX_PROCESSES);

    double index=10;
    double turnAroundTime = avgTurnAroundTime(process, MAX_PROCESSES);
    double throughPut = (double)MAX_PROCESSES / cTime;
    double cpuUti = cTime / (MAX_PROCESSES * cpuSTime);
    double diskUti = (cTime - (MAX_PROCESSES * cpuSTime)) / (MAX_PROCESSES * diskSTime);

    
    cout << "Turn Around Time: " << turnAroundTime/10 << endl;
    cout << "Throughput: " << throughPut*100 << endl;
    cout << "CPU Utilization: " << cpuUti*10 << "%" << endl;
    cout << "Disk Utilization: " << diskUti*10 << "%"  << endl;
    cout << "Average Processes in CPU: " << avgProcInCPU(lambda) << endl;
    cout << "Average Processes in Disk: " << avgProcInDisk(lambda) << endl;
    cout << "Clock: " << cTime<<endl;

    ofstream fout;
    fout.open("Output.txt");
        for(int x=0; x<MAX_PROCESSES; x++)
        {
            fout << "ID: " << process[x].ID<< " Arrival time: "<< process[x].arrivalTime<< " Completion time: "<< process[x].completionTime<<endl; 
        }
    fout.close();

    return 0;
}

double avgProcInCPU(double lambda) 
{
    return 1/lambda;                //Not done
}

double avgProcInDisk(double lambda) 
{
    return 1/lambda;                //Not done
}


double avgTurnAroundTime(Process process[], int MAX_PROCESSES)
{
    double totalTurnaroundTime = 0;

    for (int i = 0; i < MAX_PROCESSES; i++) {
        if (process[i].complete) {
            double turnaroundTime = process[i].completionTime - process[i].arrivalTime;
            totalTurnaroundTime += turnaroundTime;
        }
    }

    double avgTurnAroundTime = totalTurnaroundTime / MAX_PROCESSES;
    return avgTurnAroundTime;
}

double avgServiceTime(double cpuSTime)             //Calculates random avg time around the given CPU sevice time
{
    double serviceTime;
    double range=1;
    double randNum = (double)rand() / RAND_MAX * range * 2 - range;
    serviceTime = cpuSTime + randNum;
    return serviceTime;
}

double avgServiceTimeDisk(double diskSTime)             //Calculates random avg time around the given Disk sevice time
{
    double serviceTime;
    double range=1;
    double randNum = (double)rand() / RAND_MAX * range * 2 - range;
    serviceTime = diskSTime + randNum;
    return serviceTime;
}

 void wait()                                       //Makes the queue wait before sending to disk/cpu
{
    for (int x=0; x<1; x++)
    {
        //wait
    }
}

void readyQCPU(double cpuSTime, double diskSTime, Process& p) 
{
    if (p.arrivalTime==-1)
    {
        p.arrivalTime=cTime;                            //Clock in the arrival time of this process
    }
    rQCPU.push(p);
    do
    {
        wait();
    }while ((diskState==true));
    CPU(cpuSTime, diskSTime, p);
}

double CPU(double cpuSTime, double diskSTime, Process& p) 
{
    cpuState=true;                                  //Making the CPU into a busy state
    rQCPU.pop();                                    
    double randNum = (double)rand() / RAND_MAX;

    
    if (randNum <= .4) 
    {
        cTime += cpuSTime;
        cpuState=false;                             //Making the CPU state back to idle
        cTime=cTime+avgServiceTime(cpuSTime);
        readyQDisk(cpuSTime, diskSTime, p);
    }
    else 
    {
        p.complete = true;                          //Process marked as complete 
        cpuState=false;                             //Making the CPU state back to idle
        cTime=cTime+avgServiceTime(cpuSTime);       
        p.completionTime=cTime;
        p.complete=true;
        terminateCount();
    }
    return 0;
}

void readyQDisk(double cpuSTime, double diskSTime, Process& p) 
{
    rQDisk.push(p);
    do
    {
        wait();
    } while ((cpuState==true));
    disk(cpuSTime, diskSTime, p);    
}

void disk(double cpuSTime, double diskSTime, Process& p) 
{
    diskState=true;                                 //Making the disk state into a busy state
    rQDisk.pop();
    cTime=cTime+avgServiceTimeDisk(diskSTime);
    diskState=false;                                //Making the disk state back to idle
    readyQCPU(cpuSTime, diskSTime, p);
}

int terminateCount() 
{
    done++;
    return done;
}

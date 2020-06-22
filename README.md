//用github機測 
//********************************************************************************/ 
// DS2ex05_18_10727217陳炯瑋_10727223陳宇呈 
//********************************************************************************/ 

#include <iostream>
#include <fstream>
#include <string>
#include <cstdlib>
#include <iomanip>
#include <cstring>
#include <vector>
#include <queue>
#include <stack>
#include <algorithm>

using namespace std;

#define MAX_LEN 10
#define PAGE 25
#define NONE -1

typedef struct sP {
	char putID[MAX_LEN] ; // 發訊者 
	char getID[MAX_LEN] ; // 收訊者
	float wgt ; // 權重 
} studentPair ;

typedef struct aLN {
	char getID[10] ; // 收訊者
	float weight ; // 權重 
	struct aLN * next ; // pointer to the next node
	bool visited ;
} adjListNode ;

typedef struct aL{
	char putID[10] ; // 發訊者 
	adjListNode * head ; // pointer to the first node of a list
	bool visited ;
} adjList;

typedef struct taLN {
	char getID[10] ; // 收訊者
	float weight ; // 權重 
} tempAdjList ;

typedef struct Information {
	vector<char*> id ;
	vector<float> weight ;
	vector<int> visit ;
}Information;

vector<studentPair> student ;
vector<adjList> adjL ; // the adjacency lists
vector<Information> idList ;

class mission0 {
	vector<char*> count ;
	
	public:
			
 	void Init() {
		student.clear() ;
		adjL.clear() ;	
		count.clear() ;
	} // Init
		
	// **********************************讀檔*******************************************// 
	
    void readBinary(string filename){
    	string tempfile_name = filename ;
		tempfile_name = "pairs" + filename + ".bin" ;
		fstream binFile;
		studentPair oneSp;
		int stNo = 0;	
		binFile.open(tempfile_name.c_str(),fstream::in |fstream::binary);	
		if(binFile.is_open()){
			binFile.seekg(0,binFile.end);
			stNo = binFile.tellg()/sizeof(oneSp);
			binFile.seekg(0,binFile.beg);
		
			for(int i =0; i < stNo;i++){
				binFile.read((char*)&oneSp,sizeof(oneSp));
				student.push_back(oneSp) ;
			} // for	
			
			
		} // if
		else{
			cout << "### " << tempfile_name << " does not exist! ###" << endl ;
		} // else
		binFile.close();
    } // readBinary
	
	// *****************************************************************************// 
	
	void Graph(string filename, float inputwgt ) {
		fstream outfile ;
		filename = "pairs" + filename + ".adj" ;
		outfile.open((filename).c_str(), ios::out) ;  
		bool sameputID = false ;
		bool samegetID = false ;
		int i = 0 ;
		adjListNode * temp = NULL ;
		while ( i < student.size() ) {
			sameputID = false ;
			samegetID = false ;
			
			if ( student[i].wgt <= inputwgt ) {
				
				if ( count.empty() ) {
					count.push_back( student[i].putID ) ;
					count.push_back( student[i].getID ) ;
				} // if
				else {
					for ( int j = 0 ; j < count.size() && !sameputID ; j++ ) {
						if ( strcmp( count[j], student[i].putID ) == 0 ) 
							sameputID = true ;
				
					} // for
			
					if ( !sameputID ) {
						count.push_back( student[i].putID ) ;
					} // if
			
					for ( int j = 0 ; j < count.size() && !samegetID; j++ ) {
						if ( strcmp( count[j], student[i].getID ) == 0 ) 
							samegetID = true ;
	
					} // for
			
					if ( !samegetID ) {
						count.push_back( student[i].getID ) ;
					} // if
			
				} // else
				
			} // if
			
			i++ ;
		} // while
		
		for ( int i = 0 ; i < count.size() ; i++ ) {
			for ( int j = 0 ; j < count.size() ; j++ ) {
				if ( strcmp( count[i], count[j] ) < 0 )
					swap( count[i], count[j] ) ;
			} // for
		} // for
		
		i = 0;
		adjList temp2 ;
		while( i < count.size() ){
			strcpy( temp2.putID, count[i] ) ;
			adjL.push_back( temp2 ) ;
			insert(temp2, i, inputwgt);
			i++ ;
		} // while
		
		int num = 0 ;
		int nodenum = 0 ;
		outfile << 	"<<< There are "<< adjL.size() << " IDs in total. >>>" << '\n' ;
		for ( int i = 0 ; i < adjL.size() ; i++ ) {
				num = 1 ;
				adjListNode * run = adjL[i].head ;
				outfile << "[  " << i+1 << "] " << adjL[i].putID << ": " << '\n' ; 
				while( run != NULL ) {
					if ( num % 10 == 1 && num != 1 )
						outfile << '\n' ;
					
					outfile << '\t' << "(" << num << ") " ;									
					outfile << run->getID << ",  " << run->weight ;
					run = run -> next;
					num++;
					nodenum++ ;
				} // while
				
				outfile << '\n' ;
				
		} // for
		
		outfile << "<<< There are " << nodenum <<" nodes in total. >>>" << '\n' ;
		
		cout << "<<< There are "<< adjL.size() << " IDs in total. >>>" << '\n' ;
		cout << "<<< There are " << nodenum <<" nodes in total. >>>" << '\n' ;
		
	} // Graph
	
	void insert( adjList &adj, int site, float inputwgt ){
		adjListNode *head = NULL;
		adjListNode *run = NULL;
		adjListNode *now = NULL;
		tempAdjList temp;
		vector<tempAdjList> get;
		get.clear() ;
		int i = 0;
		for( i = 0 ; i < student.size() ; i++ ) {   //  全部資料跟count做比較 
			if( strcmp(adj.putID, student[i].putID) == 0 ) {
				if ( student[i].wgt <= inputwgt ) {
					strcpy(temp.getID, student[i].getID ) ;
					temp.weight = student[i].wgt ;
					get.push_back(temp);
				} // if
			} // if
			
			if( strcmp(adj.putID, student[i].getID) == 0 ) {
				if ( student[i].wgt <= inputwgt ) {
					strcpy(temp.getID, student[i].putID ) ;
					temp.weight = student[i].wgt ;
					get.push_back(temp);
				} // if
			} // if
			
		} // for
		

		for ( int i = 0 ; i < get.size() ; i++ ) {   //排序 
			for ( int j = 0 ; j < get.size() ; j++ ) {
				if ( strcmp( get[i].getID, get[j].getID ) < 0 )
					swap( get[i], get[j] ) ;
			} // for
		} // for
		

			
		if( !get.empty() ) {
			
			run = new adjListNode(); 
			head = run ;
			strcpy( run->getID, get[0].getID );
			run->weight = get[0].weight;
			run ->next=NULL;
			now = run;
		
			
			for ( i = 1; i < get.size() ; i++ ){
				run = new adjListNode(); 
				strcpy( run->getID, get[i].getID );
				run->weight = get[i].weight;
				run ->next = NULL;
				now->next = run ;
				now = now->next ;
				
				
			}
		
		}
		else{
			head = NULL;
		}
		
		adjL[site].head = head ;
		
	} // insert	
	
};

class mission1 {
	public:	
	
	void DFS(string filename, string inputweight) {
		Information temp ;
		temp.id.clear() ;
		idList.clear();
		for ( int i = 0 ; i < adjL.size() ; i++ ) { // 先將所有點設為false 
			adjL[i].visited = false ;
			adjListNode * run  = adjL[i].head ;
			while ( run != NULL )	{
				run->visited = false ;
				run = run->next ;
			} // while
		} // for
		
		for ( int j = 0 ; j < adjL.size() ; j++ ) {
			temp.id.clear() ;
			
			if ( adjL[j].visited == false ) {				
				temp.id.push_back( adjL[j].putID ) ;
				Recursion( adjL, j, temp ) ;
				idsort(temp);
				idList.push_back( temp ) ;
			} // if
		} // for
				
		sort(idList);
		printinformation(filename , idList, inputweight);
	} // DFS
	
	void idsort(Information &temp){
		for ( int i = 0 ; i < temp.id.size() ; i++ ) { 
			for ( int j = 0 ; j < temp.id.size() ; j++ ) {
				if ( strcmp( temp.id[i], temp.id[j] ) < 0 ) {
					swap( temp.id[i], temp.id[j] ) ;
				} // if
			} // for
		} // for
	}
	
	void sort(vector<Information> &temp){
		for ( int i = 0 ; i < temp.size() ; i++ ) { 
			for ( int j = 0 ; j < temp.size() ; j++ ) {
				if ( temp[i].id.size() > temp[j].id.size() ) {
					swap( temp[i], temp[j] ) ;
				} // if
				else if ( temp[i].id.size() == temp[j].id.size() ){
					if ( strcmp(temp[i].id[0] , temp[j].id[0]) > 0 )
						swap( temp[i], temp[j] ) ;				
				} // else if
			} // for
		} // for
	}
	
	void Recursion( vector<adjList> & adjL, int site, Information & store ) {
		
		for ( int i = 0 ; i < adjL.size() ; i++ ) {  // 將全部跑過有這個數的設為true 
			if ( strcmp( adjL[i].putID, adjL[site].putID ) == 0 ) {
				adjL[i].visited = true ;	
				
			} // if
			
			for ( adjListNode * run = adjL[i].head ; run != NULL ; run = run->next ) {
				if ( strcmp( adjL[site].putID, run->getID ) == 0 ) {
					run->visited = true ;
				} // if
				
			} // for
		} // for
		
		for ( adjListNode * head = adjL[site].head ; head != NULL ; head = head->next ) { 
			if ( head->visited == false ) {
				for ( int i = 0 ; i < adjL.size() ; i++ ) {
					if ( strcmp( adjL[i].putID, head->getID ) == 0 ) {
						store.id.push_back( head->getID ) ;
						
						Recursion( adjL, i, store ) ;
					} // if
				} // for
			} // if 
		} // for
		
	} // Recursion
	
	void printinformation(string filename , vector<Information> &idList, string inputweight) {
		int num = 0;
		fstream out ;
		
		filename = "pairs" + filename + "_" + inputweight + ".cc" ;
		out.open((filename).c_str(), ios::out) ;
		if ( out ) {
			out << "<<< There are " << idList.size() << " connected components in total. >>>" << endl ;
			for ( int i = 0 ; i < idList.size() ; i++ ) {
				num = 0 ;
				out << "{ "<< i+1 << "} " << "Connected Component: size = " << idList[i].id.size() << endl ;
				for ( int m = 0 ; m < idList[i].id.size() ; m++ ) {
					if ( ( num % 8 ) == 0 && num != 0 ) 
						out  << '\n' << '\t' ;
					if ( num == 0 ) 
						out << '\t' ;
					if ( num < 9)
						out << "(  " <<  m + 1 << ") " << idList[i].id[m] << "  " ;
					else if ( num < 99)
						out << "( " <<  m + 1 << ") " << idList[i].id[m] << "  " ;
					else
						out << "(" <<  m + 1 << ") " << idList[i].id[m] << "  " ;
					num++ ;
				} // for
			
				out << '\n' ;
				if ( ( num % 8 ) == 0 ) out << '\n' ;
			} // for
				
		} // if	
			
		cout << endl <<"<<< There are " << idList.size() << " connected components in total. >>>" << endl ;
		for ( int i = 0 ; i < idList.size() ; i++ ) {
			cout << "{ "<< i+1 << "} " << "Connected Component: size = " << idList[i].id.size() << endl ;	
		}
	} // printinfromation()
	
	void printa() {
		for ( int i = 0 ; i < adjL.size() ; i++ ) {
			adjListNode * run  = adjL[i].head ;
			cout << "[  " << i+1 << "] " << adjL[i].putID << ": " << '\n' ; 
			while ( run != NULL ) {
				cout << run->getID << ",  " << run->weight ;
				run = run -> next;
			} // while
			cout << "\n" ;
		} // for
	} // void
	

};


class mission2{
	public:
	void findposition(int &position, int idsite, int site){//找在adjL的位置 
		bool test = false;
		
		for( int i = 0 ; !test && i < idList[site].id.size() ; i++ ){//把 idlist的每個學號在adjL的位置放入position
			for( int j = 0 ; j < adjL.size() ; j++ ){
				if( adjL[j].putID == idList[site].id[idsite] ){
					position = j;
					test = true;
					
					break;
				}
			}
		}
	}
	
	void minstep( string inputweight, char inputID[10], string filename ) {
		int i = 0, j= 0,test = 0;
		int site = -1, idsite = -1 ;
		int position = -1, min = 0,minnext = 0;
		int zero = 0;
		adjListNode * temp;
		
		for (i = 0; site == -1 && i < idList.size() ; i++ ){  //找到學號位置(idlist)
			for( j =0; j < idList[i].id.size() ; j++){
					if( strcmp( inputID , idList[i].id[j] ) == 0 ){  // 找到學號在idList中的位置 
						site = i;
						break;
					}
			}
		}
		
		//site--;
		
		idsite = j;
		float one =-1;
		for( i = 0; i < idList[site].id.size() ; i++)
			idList[site].weight.push_back(one) ;
			
			
		findposition(position, idsite, site);
		
		min = idsite;
		for( i = 0 ; i < idList[site].id.size() ; i++ ){
			
			idList[site].visit.push_back( zero );
			
			if( strcmp( idList[site].id[i] ,inputID ) == 0 ){
				
				idList[site].visit[i] = 1;
				idList[site].weight[i] = 0;
				continue;
			}
			
			
			for( temp = adjL[position].head ; temp != NULL ; temp = temp->next ){	
			  if (strcmp(adjL[position].head->getID, idList[site].id[i] ) == 0)
			    break ;	
		      
			}
			
			if( temp == NULL)
				idList[site].weight[i]  = -1;
			else{  //中間有找到ID 
				
				idList[site].weight[i] = temp->weight;
				cout << idList[site].weight[i] << " gkgf"<<endl;
				if( idList[site].visit[min] == 1 && idList[site].visit[i] == 0  ) { // 第一次設定 
					min = i;
				}
				else if ( idList[site].visit[min] == 0 && idList[site].visit[i] == 0 && idList[site].weight[i] < idList[site].weight[min] ) { // 調整最小路徑 
					min = i; 
				}
			}	
		}
		
		
		idList[site].visit[min] = 1; //  設定為起點 
		minnext = min;//設定下一次的起點 
		while( test == 0){
			
			findposition(position, min, site );
			for(temp = adjL[position].head ; temp != NULL ; temp = temp->next ){	//目前最小路徑的getID往後		
				for( i = 0 ; i < idList[site].id.size() && strcmp(adjL[min].head->getID, idList[site].id[i] ) != 0 ; i++ ){// 去看是否有在idList 
				}
				if( i != idList[site].id.size() ) { 
					if( idList[site].weight[i] > idList[site].weight[min] + temp->weight || idList[site].weight[i] == -1 ){ // 更改最短距離 
						cout << "in";
						idList[site].weight[i] =  idList[site].weight[min] + temp->weight;
					}
					if( idList[site].visit[minnext] == 1 && idList[site].visit[i] == 0 ){  //第一個先當新起點 
						minnext = i;
					} 
					else if( idList[site].visit[minnext] == 0 && idList[site].visit[i] == 0 && idList[site].weight[i] < idList[site].weight[minnext] ){  //找新起點 
						minnext = i;
					}
				}
			}
			if( idList[site].visit[minnext] == 1 ){  //全都走訪過了->>>>結束 
				test = 1;
			}
			else{
				idList[site].visit[minnext] = 1;//走訪過了 
			}
			min = minnext;
		}
		
		test = 0;
		while( test == 0 ){
			
			for( i = 0; i < idList[site].id.size(); i++ ){
				test = 1;
				if( idList[site].weight[i] == -1 ){   //還是有沒跑到 
					test = 0;
					for( j = 0; j < idList[site].id.size(); j++ ){
						findposition(position, j, site );
						for( temp = adjL[position].head ; temp != NULL && strcmp(temp->getID , idList[site].id[i]) != 0 ; temp = temp->next ){							
						}
						if(temp != NULL && idList[site].weight[j] != -1 ){
							if( idList[site].weight[i] == -1 ){
								idList[site].weight[i] = idList[site].weight[j] + temp->weight;
							}
							else if( idList[site].weight[i] > idList[site].weight[j] + temp->weight){
								idList[site].weight[i] = idList[site].weight[j] + temp->weight;
							}	
						}
					}
					if( idList[site].weight[i] != -1 )
						idList[site].visit[i] = 1;
				}
			}
		}
		
		weightsort(site);
		printmis2information( inputID, inputweight, site , filename);
	}
	
	void weightsort( int site ){
		for( int i = 0 ; i < idList[site].id.size() ; i++ ){
			for( int j = 0 ; j < idList[site].id.size() ; j++ ){
				if( idList[site].weight[i] < idList[site].weight[j] ){
					
					swap( idList[site].weight[i] , idList[site].weight[j] );
					swap( idList[site].id[i] , idList[site].id[j] );
					
				}
			}
		}
	}
	
	
	
	
	void printmis2information( char inputID[10], string inputweight , int site , string filename) {
		int num = 0, i = 0, j= 0;
		fstream out ;
		filename = "pairs" + filename + "_" + inputweight + ".ds" ;
		out.open((filename).c_str(), ios::out) ;
		
		if ( out ) {
			out << "origin: " << inputID << "\n" ;
			for( i = 0; i < idList[site].id.size() ; i++ ){
				if( strcmp( idList[site].id[i] , inputID) != 0){
					if ( num < 10 ){
						if( num %8 != 0 )
							out << "( " << num << ")" << "\t" << idList[site].id[i] << ", " << idList[site].weight[i] << "\t" ;
						if ( num == 9 )
							out << "( " << num << ")" << "\t" << idList[site].id[i] << ", " << idList[site].weight[i] << "\n" ;
					}
					else if ( num >= 10) {					
						if ( num % 8 != 0)
							out << "(" << num << ")" << "\t" << idList[site].id[i] << ", " <<idList[site].weight[i] << "\t" ;
						else 
						 	out << "(" << num << ")" << "\t" << idList[site].id[i] << ", " << idList[site].weight[i] << "\n" ;
					}
				}
			}
		}
		
		for ( i = 0 ; i < idList.size() ; i++ ){
			for( j = 0 ; j < idList[i].id.size() ; j++){
				cout << idList[i].id[j] << "\t";
			}
		}
	}
};




int main(int argc, char** argv) {
	int num = 0 ,i =0;
	bool done = false ;
	string filename ;
	string inputweight;
	float inputwgt = 0 ;
	mission0 mis0 ;
	mission1 mis1 ;
	mission2 mis2 ;
	char  inputID[10];
	vector<Information> idList;
	do {
		cout << "******* Graph data applications ******" << endl ;
		cout << "* [Any other key: QUIT]              *" << endl ;
		cout << "* 0. Create adjacency lists          *" << endl ;
		cout << "* 1. Build connected components      *" << endl ;
		cout << "* 2. Find shortest paths by Dijkstra *" << endl ;
		cout << "**************************************" << endl ;	
		cout << "Input a choice(0, 1, 2) [Any other key: QUIT]:"  ;
		
		cin >> num ;
		if ( num == 0 ) {
		    cout << "Input a real number in (0,1]:" ;
		    cin >> inputweight ;
		    inputwgt = atof(inputweight.c_str());
		    if ( inputwgt > 0 && inputwgt <= 1 ) {
		    	while(true) {
					cout << "Input a file number ([0] Quit): " ;
					cin >> filename ;
					if(!filename.compare("0"))
						return false ;
					else
						break ;
				} // while
		    	
		    	mis0.Init() ;
		    	mis0.readBinary( filename ) ;
		    	mis0.Graph( filename, inputwgt ) ;
			} // if 
			else {
				cout << "### It is NOT in (0,1] ###" ;
			} // else
		} // if
		else if ( num == 1 ) {
			mis1.DFS(filename, inputweight) ;
			
		} // else if
		else if ( num == 2 ) {
			for( i = 0 ; i < adjL.size() ; i++ ){
				cout << adjL[i].putID << "  ";
			}
			cin >> inputID;
			mis2.minstep( inputweight, inputID, filename );
		} // esle if
		else
			done = true ;
	
		
	} while( !done ) ;
}

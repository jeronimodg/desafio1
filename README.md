#include <iostream>
#include <cstdlib>
#include <ctime>
using namespace std;
//===== TABLERO =====

unsigned char** crearTablero(unsigned short int alto, unsigned short int anchoBytes){
    unsigned char** tablero = new unsigned char*[alto];
    for(unsigned short int i=0;i<alto;i++)
        tablero[i] = new unsigned char[anchoBytes];
    return tablero;
}

void inicializarTablero(unsigned char** tablero, unsigned short int alto,unsigned short int anchoBytes){
    for(unsigned short int i=0;i<alto;i++)
        for(unsigned short int j=0;j<anchoBytes;j++)
            tablero[i][j] = 0;
}

void liberarTablero(unsigned char** tablero,  unsigned short int alto){
    for( unsigned short int i=0;i<alto;i++)
        delete[] tablero[i];
    delete[] tablero;
}

// ===== PIEZAS =====

int generarPieza(){
    return rand() % 7;
}

void obtenerPieza(unsigned short int tipo, unsigned short pieza[4]){
    for(unsigned short int i=0;i<4;i++) pieza[i]=0;
    switch(tipo)
    {
    case 0: pieza[1]=0b1111; break;
    case 1: pieza[1]=0b0110; pieza[2]=0b0110; break;
    case 2: pieza[1]=0b1110; pieza[2]=0b0100; break;
    case 3: pieza[1]=0b0110; pieza[2]=0b1100; break;
    case 4: pieza[1]=0b1100; pieza[2]=0b0110; break;
    case 5: pieza[1]=0b1000; pieza[2]=0b1110; break;
    case 6: pieza[1]=0b0010; pieza[2]=0b1110; break;
    }
}
void rotarPieza(unsigned short pieza[4]){
    unsigned short nueva[4]={0,0,0,0};
    for(unsigned short int i=0;i<4;i++)
        for(unsigned short int j=0;j<4;j++)
            if(pieza[i] & (1<<j))
                nueva[j] |= (1<<(3-i));
    for(unsigned short int i=0;i<4;i++)
        pieza[i]=nueva[i];
}

// ===== VALIDACIÓN =====

bool validacionFronteras(
    unsigned char** tablero,
    unsigned short int alto,
    unsigned short int anchoBytes,
    unsigned short pieza[4],
    unsigned short int fila,
    unsigned short int columna
    ){
    unsigned short int anchoTotal = anchoBytes * 8;
    for(unsigned short int i=0;i<4;i++){
        if(pieza[i]==0) continue;
        unsigned short int f = fila + i;
        if(f >= alto) return false;
        for(unsigned short int bit=0; bit<16; bit++){
            if(pieza[i] & (1<<bit))
            {
                short int colReal = columna + bit;
                if(colReal < 0 || colReal >= anchoTotal)
                    return false;
                if(f >= 0)
                {
                    unsigned short int byte = colReal / 8;
                    short int bitReal = 7 - (colReal % 8);
                    if(tablero[f][byte] & (1<<bitReal))
                        return false;
                }
            }
        }
    }
    return true;
}

// ===== FUSIÓN =====

void fusionarPiezaConTablero(
    unsigned char** tablero,
    unsigned short pieza[4],
    unsigned short int fila,
    unsigned short int columna,
    unsigned short int anchoBytes
    ){
    for(unsigned short int i=0;i<4;i++){
        if(pieza[i]==0) continue;

        short int f = fila+i;
        if(f < 0) continue;

        for(unsigned short int bit=0; bit<16; bit++)
        {
            if(pieza[i] & (1<<bit))
            {
                unsigned short int colReal = columna + bit;
                unsigned short int byte = colReal / 8;
                short int bitReal = 7 - (colReal % 8);
                tablero[f][byte] |= (1<<bitReal);
            }
        }
    }
}

// ===== FILAS =====

bool filaLlena(unsigned char* fila, unsigned short int anchoBytes){
    for(unsigned short int i=0;i<anchoBytes;i++)
        if(fila[i] != 255)
            return false;
    return true;
}
int eliminarFilas(unsigned char** tablero, unsigned short int alto, unsigned short int anchoBytes){
    unsigned short int filasEliminadas = 0;
    for(short int i=alto-1;i>=0;i--){
        if(filaLlena(tablero[i], anchoBytes))
        {
            filasEliminadas++;
            for(unsigned short int k=i;k>0;k--)
                for(unsigned short int j=0;j<anchoBytes;j++)
                    tablero[k][j] = tablero[k-1][j];
            for(unsigned short int j=0;j<anchoBytes;j++)
                tablero[0][j] = 0;
            i++;
        }
    }
    return filasEliminadas;
}

// ===== GAME OVER =====

bool gameOver(unsigned char** tablero, unsigned short int anchoBytes, char mov){
    if(mov=='q' || mov=='Q'){ 
        return true;
    }
    for(unsigned short int j=0;j<anchoBytes;j++)
        if(tablero[0][j]!=0)
            return true;
    return false;
}

// ===== DIBUJO =====

void imprimirConPieza(
    unsigned char** tablero,
    unsigned short int alto,
    unsigned short int anchoBytes,
    unsigned short pieza[4],
    short int fila,
    unsigned short int columna,
    unsigned short int puntaje
    ){
    cout << "\nPUNTAJE: " << puntaje << "\n";
    for(unsigned short int i=0;i<alto;i++){
        cout<<"|";
        for(unsigned short int j=0;j<anchoBytes;j++){
            for(short int bit=7;bit>=0;bit--){
                bool ocupado = false;
                if(tablero[i][j] & (1<<bit))
                    ocupado = true;
          
                for(short int k=0;k<4;k++)
                {
                    short int f = fila + k;
                    
                    if(f != i || f < 0) continue;
                    
                    for(unsigned short int b=0;b<16;b++){
                        if(pieza[k] & (1<<b)){
                            unsigned short int colReal = columna + b;
                            unsigned short int byte = colReal / 8;
                            unsigned short int bitReal = 7 - (colReal % 8);
                            if(byte==j && bitReal==bit)
                                ocupado = true;
                        }
                    }
                }
                cout<<(ocupado ? "#" : ".");
            }
        }
        cout<<"|\n";
    }

    cout<<"\n";
}

// ===== MAIN =====


int main(){
    unsigned short int ancho=0, alto=0;

    while(ancho%8!=0 || ancho<8 || alto<8){
        cout<<"Ancho (multiplo de 8): ";
        cin>>ancho;
        cout<<"Alto (>=8): ";
        cin>>alto;
    }
    unsigned short int anchoBytes = ancho/8;
    unsigned char** tablero = crearTablero(alto, anchoBytes);
    inicializarTablero(tablero, alto, anchoBytes);
    srand(time(0));
    unsigned short int puntaje = 0;
    char mov='x';
    while(!gameOver(tablero, anchoBytes, mov)){
        
        unsigned short pieza[4];
        unsigned short int tipo = generarPieza();
        obtenerPieza(tipo, pieza);
        short int fila = -2;
        short int columna = ancho/2 - 2;
        bool activa = true;
        
        while(activa){
            imprimirConPieza(tablero,alto,anchoBytes,pieza,fila,columna,puntaje);
            
            cout<<"[a]izq [d]der [s]bajar [w]rotar [q]salir: ";
            cin>>mov;
            
            if(mov=='a' || mov=='A')
            {
                short int nuevaCol = columna - 1;
                if(validacionFronteras(tablero,alto,anchoBytes,pieza,fila,nuevaCol))
                    columna = nuevaCol;
                else
                    cout<<"Movimiento invalido\n";
            }
            else if(mov=='d' || mov=='D')
            {
                unsigned short int nuevaCol = columna + 1;
                if(validacionFronteras(tablero,alto,anchoBytes,pieza,fila,nuevaCol))
                    columna = nuevaCol;
                else
                    cout<<"Movimiento invalido\n";
            }
            else if(mov=='w' || mov=='W')
            {
                unsigned short copia[4];
                for(unsigned short int i=0;i<4;i++) copia[i]=pieza[i];

                rotarPieza(pieza);

                if(!validacionFronteras(tablero,alto,anchoBytes,pieza,fila,columna))
                {
                    for(unsigned short int i=0;i<4;i++) pieza[i]=copia[i];
                    cout<<"Movimiento invalido\n";
                }
            }
            else if(mov=='s' || mov=='S')
            {
                unsigned short int nuevaFila = fila + 1;

                if(validacionFronteras(tablero,alto,anchoBytes,pieza,nuevaFila,columna))
                {
                    fila = nuevaFila;
                }
                else
                {
                    fusionarPiezaConTablero(tablero,pieza,fila,columna,anchoBytes);

                    unsigned short int eliminadas = eliminarFilas(tablero,alto,anchoBytes);
                    puntaje += eliminadas * 50;

                    activa = false;
                }
            }
            else if(mov=='q' || mov=='Q'){
                activa = false;               
            }
            else
            {
                cout<<"Comando invalido\n";
            }
        }
    }
    liberarTablero(tablero, alto);
    cout<<"GAME OVER\n";

    

    return 0;
}

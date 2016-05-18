
//
//  main.cpp
//  project 5 2
//
//  Created by Wendy Xie on 5/15/16.
//  Copyright © 2016 CS 31. All rights reserved.
//

#include <iostream>
#include <cctype>
#include <string>
#include <fstream>

using namespace std;
int format(int lineLength, istream& inf, ostream& outf);
void cout_out_array(int begin, int end, char temp[], ostream& outf);
void getCharacter (istream& inf, char& c, bool& isGetCharacter);
void processingToken (char TEMP[], int& lineLength, int& token, int& position, bool& superLongToken, bool& endOfStc, bool& paragraphBreakBefore, bool& normalWords, bool& wordPortion, bool& space, bool& isGetCharacter, int& countNonSpaceToken, ostream& outf);

int main(){
    
    ifstream infile("/Users/Windy/Documents/UCLA/College/UCLA 2016 spring/CS 31/code/Project 5 2/data.txt");
    if ( ! infile )
    {
        cerr << "Error: Cannot open data.txt!" << endl;
    }
    
    ofstream outfile ("/Users/Windy/Documents/UCLA/College/UCLA 2016 spring/CS 31/code/Project 5 2/results.txt");
    if (! outfile)
    {
        cerr << "Error: Cannot create results.txt!" << endl;
    }

    cout << "Enter maximum line length: ";
    int len;
    cin >> len;
    cin.ignore(10000, '\n');
    
    int returnCode = format(len, infile, outfile);
    
    outfile << endl << "Return code is " << returnCode << endl;
    
}

int format(int lineLength, istream& inf, ostream& outf)
{
    
    if (lineLength < 1)
        return 2;
    
    char c = '\0';
    
    char TEMP[151];
    int token = 0;                              //the length of the token that is being processed
    int position = 0;                           //the position of the character that is being processed
    bool superLongToken = false;                //mark the token which is longer than the line length
    bool endOfStc = false;                      //mark the end of the sentence
    bool paragraphBreakBefore = false;          //mark the "#P#"
    bool normalWords = false;                   //mark the normal words without '.' '?' and '-'
    bool wordPortion = false;                   //mark the word token with '-' at the end
    bool space = false;                         //mark if the space
    bool isGetCharacter = true;                 //mark the end of the text
    int countNonSpaceToken = 0;                 //count the number of token that is not a space


    for(;;)
    {
        token = 0;
        while(isGetCharacter)
        {
            getCharacter(inf, c, isGetCharacter);
            if (isGetCharacter && isspace(c))
            {
                TEMP[token] = '\0';
                break;
            }
            else if (isGetCharacter && c == '-')
            {
                TEMP[token] = c;
                token ++;
                TEMP[token] = '\0';
                break;
            }
            else if(isGetCharacter)
            {
                TEMP[token] = c;
                token ++;
                TEMP[token] = '\0';
            }
        }
        
        if (TEMP[0] == '\0' && !isGetCharacter)
        {
            if (superLongToken)
                return 1;
            else
                return 0;
        }
        
        if (TEMP[0] != '\0' && !(TEMP[0]=='#' && TEMP[1]=='P' && TEMP[2]=='#'))
        {
            countNonSpaceToken ++;
        }
        
        if (countNonSpaceToken == 1)
        {
            if (TEMP[0]=='#' && TEMP[1]=='P' && TEMP[2]=='#')
                paragraphBreakBefore = true;
            else if (TEMP[token] == '\0' && (TEMP[token-1] =='.' || TEMP[token-1] == '?'))
                endOfStc = true;
            else if (TEMP[token] == '\0' && (TEMP[token-1] == '-'))
                wordPortion = true;
            else
                normalWords = true;
            if (TEMP[0] == '\0')
                space = true;
        }
        
        if (TEMP[0] == '\0')
        {
            space = true;
            if (countNonSpaceToken == 0)
                space = false;
        }
        else if (TEMP[0]=='#' && TEMP[1]=='P' && TEMP[2]=='#')
        {

            endOfStc = false;
            paragraphBreakBefore = true;
            normalWords = false;
            wordPortion = false;
            space = false;
            position = 0;
        }
        else if (TEMP[token] == '\0' && (TEMP[token-1] =='.' || TEMP[token-1] == '?' ))
        {
            processingToken(TEMP, lineLength, token, position, superLongToken, endOfStc, paragraphBreakBefore, normalWords, wordPortion, space, isGetCharacter, countNonSpaceToken, outf);
            
            endOfStc = true;
            paragraphBreakBefore = false;
            normalWords = false;
            wordPortion = false;
            space = false;
        }
        
        else if (TEMP[token] == '\0' && (TEMP[token-1] == '-'))
        {
            processingToken(TEMP, lineLength, token, position, superLongToken, endOfStc, paragraphBreakBefore, normalWords, wordPortion, space, isGetCharacter, countNonSpaceToken, outf);
            
            endOfStc = false;
            paragraphBreakBefore = false;
            normalWords = false;
            wordPortion = true;
            space = false;
        }

        else
        {
            processingToken(TEMP, lineLength, token, position, superLongToken, endOfStc, paragraphBreakBefore, normalWords, wordPortion, space, isGetCharacter, countNonSpaceToken, outf);
            
            endOfStc = false;
            paragraphBreakBefore = false;
            normalWords = true;
            wordPortion = false;
            space = false;
        }
        
        if (!isGetCharacter)
        {
            if (superLongToken)
                return 1;
            else
                return 0;
        }
    }
    return 0;
}

void cout_out_array(int begin, int end, char temp[], ostream& outf)
{
    for (int k = begin; k <= end; k++)
        outf << temp[k];
}

void getCharacter (istream& inf, char& c, bool& isGetCharacter)
{
    if (inf.get(c))
        isGetCharacter = true ;
    else
        isGetCharacter = false;
}

void processingToken (char TEMP[], int& lineLength, int& token, int& position, bool& superLongToken, bool& endOfStc, bool& paragraphBreakBefore, bool& normalWords, bool& wordPortion, bool& space, bool& isGetCharacter, int& countNonSpaceToken, ostream& outf)
{
    if (endOfStc && (position+token +2 <= lineLength))
    {
        if(countNonSpaceToken != 1 && isGetCharacter)
        {
            outf << "  ";
            position += 2;
        }
        
        position += token;
        cout_out_array(0, token-1, TEMP, outf);
    }
    
    else if (normalWords && (position + token + 1 <= lineLength))
    {
        if (countNonSpaceToken !=1)
        {
            outf << " ";
            position ++;
        }
        position += token;
        cout_out_array(0, token-1, TEMP, outf);
    }
    else if (wordPortion && (position+token <= lineLength))
    {
        if (space)
        {
            if (position + token + 1 <= lineLength && isGetCharacter)
            {
                outf << " ";
                position ++;
            }
            else
            {
                outf << "\n";
                position = 0;
            }
        }
        
        position += token;
        cout_out_array(0, token-1, TEMP, outf);
    }
    
    else if (paragraphBreakBefore)
    {
        if(countNonSpaceToken != 1 && isGetCharacter)
        {
            outf << "\n\n";
        }
        position += token;
        cout_out_array(0, token-1, TEMP, outf);
    }
    
    else if (token > lineLength)
    {
        if(countNonSpaceToken != 1)
        {
            outf << "\n";
        }
        for (int k = 0; k <= token; k += 1)
        {
            outf << TEMP[k];
            if (k != token && (k+1)%lineLength == 0)
            {
                outf << "\n";
            }
            position = token - ((k/lineLength) * lineLength);
        }
        superLongToken = true;
    }
    
    else
    {
        if(countNonSpaceToken != 1)
        {
            outf << "\n";
        }
        cout_out_array(0, token-1, TEMP, outf);
        position = token;
    }
}

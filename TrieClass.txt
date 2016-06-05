from functools import *
import operator
import time

class Node:
    
    """
    A class that acts as the branches and nodes of a Trie.
    """
    
    def __init__(self):
        """
        Constructor for each node.
        
        Fields
        self.__words -- number of words that use this node as its ending 
                            character.
        self.__prefixes -- number of words that contains this node as a 
                            prefix character.
        self.__edges -- array of links to 26 letters that follow this node.
        """
        
        self.__words = 0
        self.__prefixes = 0
        self.__edges = [None] * 26
        
    @staticmethod
    def __retrieveIndex(char):
        return ord(char) - ord('a')
    
    @staticmethod
    def __retrieveChar(index):
        return chr(index + ord('a'))
    
    def getWords(self): return self.__words
    def getPrefixes(self): return self.__prefixes
    def getEdges(self): return self.__edges
        
    def insertHelper(self, wordArr): 
        
        if (len(wordArr) > 0):
            
            childNodeIndex = self.__retrieveIndex(wordArr[0])
        
            if (self.getEdges()[childNodeIndex] == None):
                newNode = Node()
                self.__edges[childNodeIndex] = newNode
            
            self.__prefixes += 1
            
            self.__edges[childNodeIndex].insertHelper(wordArr[1:])

        
        else: 
            self.__prefixes += 1
            self.__words += 1
            
    def lookupHelper(self, wordArr):
        
        if (len(wordArr) > 0):
            childNodeIndex = self.__retrieveIndex(wordArr[0])
        
            if (self.__edges[childNodeIndex] == None):
                return False
            else: 
                return self.__edges[childNodeIndex].lookupHelper(wordArr[1:])
        else: 
            if (self.getWords() > 0):
                return True
            else:
                return False
            
    def removeHelper(self, wordArr):
        if (len(wordArr) > 0):
            childNodeIndex = self.__retrieveIndex(wordArr[0])
            if (self.__edges[childNodeIndex].removeHelper(wordArr[1:]) == 1):
                self.__edges[childNodeIndex] == None
        else:
            self.__words -= 1
            
        return self.__prefixes
    
    def traversal(self, word):
        subdictionary = []
        if (self.__words > 0):
            subdictionary = subdictionary + [[''.join(list(map(lambda letter: self.__retrieveChar(letter), word))), self.__words]]
        
        for index in range(len(self.__edges)):
            if (self.__edges[index] != None):
                subdictionary = subdictionary + self.__edges[index].traversal(word + [index])
            
        return subdictionary
        
        
class MyTrie:
    
    """
    A class that contains a Trie structure for storing words in a dictionary 
    map format.
    """
    
    def __init__(self):
        """
        Trie class constructor.
        
        Arguments:
        root -- An array of length 26 which contains each starting letter in 
                    the trie.
        """
        self.__root = [None] * 26
        
    @staticmethod
    def __sanitize(word):
        return list(word.strip('/1234567890,. \n\t').lower())
    
    @staticmethod
    def __retrieveIndex(char):
        return ord(char) - ord('a')
    
    @staticmethod
    def __retrieveChar(index):
        return chr(index + ord('a'))
        
    def insert(self, word):
        """
        Method used to add words to recursively add words to the trie.
        
        Arguments:
        word -- a string to add to the trie.
        """
        if (word != ""):
            wordArr = self.__sanitize(word)
            childNodeIndex = self.__retrieveIndex(wordArr[0])

            if (self.__root[childNodeIndex] == None):
                self.__root[childNodeIndex] = Node()

            self.__root[childNodeIndex].insertHelper(wordArr[1:])
    
    def lookup(self, word):
        """
        Method used to determine if a word is contained in the trie.
        
        Arguments:
        word -- a string to lookup in the tree.
        
        Returns a boolean which tells us if 'word' is contained in the trie.
        """
        if (word != ""):
            wordArr = self.__sanitize(word)
            childNodeIndex = self.__retrieveIndex(wordArr[0])

            if (self.__root[childNodeIndex] == None):
                return False
            else:
                return self.__root[childNodeIndex].lookupHelper(wordArr[1:])
    
    def remove(self, word):
        """
        Method used to remove a word from the trie.
        
        Arguments:
        word -- a string to remove from the tree.
        """
        if (word != ""):
            wordArr = self.__sanitize(word)
            if (self.lookup(word)): # Is 'word' contained in Trie?
                childNodeIndex = self.__retrieveIndex(wordArr[0])
                if (self.__root[childNodeIndex].removeHelper(wordArr[1:]) == 1):
                    self.__root[childNodeIndex] == None # Remove link to child node.
            else:
                print(word, "is not contained in this Trie!")
                
    def dictionary(self):
        """
        Method used to create a dictionary of words from the trie.
        
        Returns an array containing each word in the trie.
        """
        dictionary = []
        
        realIndices = list(filter(lambda x: self.__root[x] != None, range(len(self.__root))))
        
        for index in realIndices:
            newWord = [index]
            nextWord = self.__root[index].traversal(newWord)
            dictionary = dictionary + nextWord
        
        return reduce(lambda x,y: x + [dictionary[:][y][0]], range(len(dictionary)), [dictionary[:][0][0]])
    
    def __dictionaryList(self, mayPrecede=operator.ge):
        numericArr = []        
        realIndices = list(filter(lambda x: self.__root[x] != None, range(len(self.__root))))
        
        for index in realIndices:
            numericArr = numericArr + self.__root[index].traversal([index])
    
        return reduce(lambda x,y: x if mayPrecede(x[1], numericArr[:][y][1]) else numericArr[:][y], range(len(numericArr)), numericArr[:][0])
    
    minElement = partialmethod(__dictionaryList, mayPrecede=operator.le)
    minElement.__name__ = 'minElement'
    minElement.__doc__ = 'Returns an element which occurs least frequently in Trie.'
    
    maxElement = partialmethod(__dictionaryList, mayPrecede=operator.ge)
    maxElement.__name__ = 'maxElement'
    maxElement.__doc__ = 'Returns an element which occurs most frequently in Trie.'

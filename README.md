<div align="center">

## subdirectories \(win32\)


</div>

### Description

It is just a function that reads subdirectories from a root into an array, and calls back user for each new directory found (i hope that you'll understand what i mean !!!!)

I "played" with pointers, it could be interesting for beginners :)

Oh, don't forget to vote !!!!!
 
### More Info
 
The root directory

I made it with Win32 functions, but it is easily usable with other systems.

Subdirs list.


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Patrick BROSSET](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/patrick-brosset.md)
**Level**          |Advanced
**User Rating**    |3.0 (9 globes from 3 users)
**Compatibility**  |C
**Category**       |[Files](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/files__3-2.md)
**World**          |[C / C\+\+](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/c-c.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/patrick-brosset-subdirectories-win32__3-1881/archive/master.zip)

### API Declarations

windows.h, malloc.h, stdio.h


### Source Code

```
/*********************
 LECTURE DE L'ARBORESCENCE DES REPERTOIRES
 -- * seeking subdirectories * --
 (WIN32 Code)
*********************/
#include <windows.h>
#include <malloc.h>
#include <stdio.h>
static void MakeFileName( char *pReturn, char *pPathName, char *pFileName );
static void AddItem( char ***pppArray, long *pArrayCount, char *pNewString );
/*******************
 Input :
 pRoot => Root directory
 ppDirs => Pointer on an array of strings (non allocated) => char **
 CallBack => User function that can decide to add or not a directory (and all of the sub dirs)
 Should return FALSE to stop the process...
 pUserParams => User parameter for the callback
 Output :
 Returns the number of subdirectories found.
 ----------
 The root directory will be the first item into the array ppDirs
*******************/
long SeekSubDirs( char *pRoot, char ***pppDirs, BOOL (*CallBack)(char *pPathName, void *pUserParam), void *pUserParam )
{
 char *pDirName;
 char *pWild;
 WIN32_FIND_DATA Find;
 HANDLE hFind;
 char **ppItems;
 long nbItems;
 long idxItem;
 BOOL bCancel = FALSE;
 // First, let's see if the root directory exists !
 pWild = (char *)malloc( strlen( pRoot ) + 10 );
 MakeFileName( pWild, pRoot, "*.*" );
 hFind = FindFirstFile( pWild, &Find );
 free( pWild );
 if( hFind == INVALID_HANDLE_VALUE ) // Should the happen if the dir. exists (even if it is empty)
 {
 if( pppDirs != NULL )
 *pppDirs = NULL;
 return( 0 );
 }
 // Let's add the root
 nbItems = 0;
 ppItems = NULL;
 AddItem( &ppItems, &nbItems, pRoot );
 if( CallBack != NULL && (*CallBack)( ppItems[ 0 ], pUserParam ) == FALSE )
 bCancel = TRUE;
 // And now, let's read each subdirectories of items
 idxItem = 0;
 while( idxItem < nbItems && bCancel == FALSE )
 {
 // Reading and add sub-directories
 pWild = (char *)malloc( strlen( ppItems[ idxItem ] ) + 10 );
 MakeFileName( pWild, ppItems[ idxItem ], "*.*" );
 hFind = FindFirstFile( pWild, &Find );
 while( hFind != INVALID_HANDLE_VALUE )
 {
 if( Find.dwFileAttributes & FILE_ATTRIBUTE_DIRECTORY && strcmp( Find.cFileName, "." ) && strcmp( Find.cFileName, ".." ) )
 {
 // Add the new dir...
 pDirName = (char *)malloc( strlen( ppItems[ idxItem ] ) + 1 + strlen( Find.cFileName ) + 1 ); // +1 for the \\ and +1 for the \0
 MakeFileName( pDirName, ppItems[ idxItem ], Find.cFileName );
 // Calls the user
 if( CallBack != NULL && (*CallBack)( pDirName, pUserParam ) == FALSE )
 {
 // Stops everything !
 bCancel = TRUE;
 }
 else
 {
 // And add the new dir.
 AddItem( &ppItems, &nbItems, pDirName );
 }
 free( pDirName );
 }
 if( FindNextFile( hFind, &Find ) == FALSE || bCancel == TRUE )
 {
 FindClose( hFind );
 hFind = INVALID_HANDLE_VALUE;
 }
 }
 free( pWild );
 // And precessing next "item"
 idxItem++;
 }
 // Ok...
 if( pppDirs != NULL && bCancel == FALSE )
 *pppDirs = ppItems;
 else
 {
 // Nothing to return, we can free everything......
 for( idxItem = 0; idxItem < nbItems; idxItem++ )
 free( ppItems[ idxItem ] );
 free( ppItems );
 if( pppDirs != NULL )
 *pppDirs = NULL;
 nbItems = 0;
 }
 return( nbItems );
}
//
//----------- Tool functions
//
//
static void MakeFileName( char *pReturn, char *pPathName, char *pFileName )
{
 strcpy( pReturn, pPathName );
 // We have to add a backslash if the last byte of the path is not one.
 if( *pReturn && *( pReturn + strlen( pReturn ) - 1 ) != '\\' )
 strcat( pReturn, "\\" );
 // and then add the filename
 strcat( pReturn, pFileName );
 // if there is a backslash at the end, we remove it...
 if( *pReturn && *( pReturn + strlen( pReturn ) - 1 ) == '\\' )
 *( pReturn + strlen( pReturn ) - 1 ) = '\0';
}
// Adds a string to a string array
static void AddItem( char ***pppArray, long *pArrayCount, char *pNewString )
{
 char **ppArray;
 // Allocates memory needed for one more item
 if( *pppArray != NULL )
 *pppArray = (char **)realloc( *pppArray, ((*pArrayCount) + 1) * sizeof( char * ) );
 else
 *pppArray = (char **)malloc( 1 * sizeof( char * ) );
 ppArray = *pppArray; // Easier to use :o)
 // Sets the new one
 ppArray[ *pArrayCount ] = malloc( strlen( pNewString ) + 1 ); // As always, don't forget the 0 at the end of the string !!!!
 strcpy( ppArray[ *pArrayCount ], pNewString );
 // And one item more into the array...
 *pArrayCount += 1;
}
/* LeChatMachine@libertysurf.fr, 6 juillet 2001 */
```


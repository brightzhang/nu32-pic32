[Index](Index.md)

# Introduction #

In this example, a word outside the program memory is used to store an unsigned int. It will start with all bits set. Each time the program is run, the word is stored in RAM, the 4k page is erased, the value is incremented, and then put back in place. So, each time the PIC is reset, the number increments, and is not lost until the PIC is reprogrammed.

First, write the code and compile, and find where the code ends in KSEG0. You can write to areas outside your code, after the next 4k aligned boundary.

You can only clear bits, so before writing to those areas all the bits must be set. This can be done 4k at a time with NVMErasePage((void **) LOCATION);**

NVMWriteWord can be used to write one 4-byte word. NVMWriteRow can write a row of 512 bytes (128 words). NVMProgram can write a multiple of 4 set of bytes.

## Code ##
```

#include <plib.h>
#include "NU32.h"

// a 4k aligned address outside the memory of this code
// search for 'Total kseg0_program_mem used' in the map file
// to see where the code ends in memory
#define NVM_PROGRAM_PAGE 0xbd008000

// number of bytes past NVM_PROGRAM_PAGE to store a word
#define NVM_WORD_LOCATION 0x400

int main(void) {
  NU32_Startup();
  NU32LED1 = 1; // off
  NU32LED2 = 1; // off

  char message[100];

  // read the word at NVM_WORD_LOCATION past NVM_PROGRAM_PAGE
  unsigned int reset_num = *(int *) (NVM_PROGRAM_PAGE + NVM_WORD_LOCATION);

  sprintf(message, "old reset num = 0x%x\r\n", reset_num);
  NU32_WriteUART1(message);

  // Erase the page of program flash starting at NVM_PROGRAM_PAGE, setting all the bits
  // you must set the bits before you are able to write them
  NVMErasePage((void *) NVM_PROGRAM_PAGE);

  // increment the number that was there
  reset_num++;

  // Write the new value
  NVMWriteWord((void*) (NVM_PROGRAM_PAGE + NVM_WORD_LOCATION), reset_num);

  sprintf(message, "new reset num = 0x%x\r\n", *(int *) (NVM_PROGRAM_PAGE + NVM_WORD_LOCATION));
  NU32_WriteUART1(message);

  // 
  while (1) {
    if (!NU32USER) {
      sprintf(message, "word = 0x%x\r\n", *(int *) (NVM_PROGRAM_PAGE + 0x200));
      NU32_WriteUART1(message);

      // delay 0.1s for debounce
      WriteCoreTimer(0);
      while (ReadCoreTimer() < 4000000);
    }
  }
  return 0;
}

```
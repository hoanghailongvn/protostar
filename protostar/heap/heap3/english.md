# **heap3**
## Source code
```
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>

void winner()
{
  printf("that wasn't too bad now, was it? @ %d\n", time(NULL));
}

int main(int argc, char **argv)
{
  char *a, *b, *c;

  a = malloc(32);
  b = malloc(32);
  c = malloc(32);

  strcpy(a, argv[1]);
  strcpy(b, argv[2]);
  strcpy(c, argv[3]);

  free(c);
  free(b);
  free(a);

  printf("dynamite failed?\n");
}
```

- The goal is to call the winner() function. As strcpy is used, it is possible to heapoverflow.

## Heap


- giải thích về chunk
- vuln trong free
- struct malloc chunk
```
struct malloc_chunk {  INTERNAL_SIZE_T      prev_size;
  INTERNAL_SIZE_T      size;

  struct malloc_chunk* fd;
  struct malloc_chunk* bk;
};
```
- unlink
```
#define unlink(P, BK, FD) {
  FD = P->fd;
  BK = P->bk;
  FD->bk = BK;
  BK->fd = FD;
}
```

- Thử fake prevsize và size với số dương thì thất bại vì trong attack string không thể có NUL
- Tìm hiểu trên mạng thấy có phương pháp -8 -4. Áp dụng luôn để ghi đè địa chỉ ret của hàm free thành địa chỉ của hàm winner(): thất bại vì unlink sẽ thay đổi code của hàm winner, sẽ bị lỗi.
- Tìm cách khác, tạo shell code để đổi flow đến winner(), lưu trong heap(a), ghi đè địa chỉ shell code lên got của puts => khi gọi puts sẽ gọi winner()

# References
https://airman604.medium.com/protostar-heap-3-walkthrough-56d9334bcd13
https://cw.fel.cvut.cz/old/_media/courses/a4m33pal/04_dynamic_memory_v6.pdf
Vì sao phải là -4: http://phrack.org/issues/57/9.html
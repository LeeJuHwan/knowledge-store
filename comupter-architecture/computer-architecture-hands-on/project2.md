# Project 2. Boolean Arithmetic

## Project2: Boolean Arithmetic

![image](../../.gitbook/assets/project2_image0001.png)

**프로젝트 1에서 작성한 모든 칩들을 이용 하여 반가산기, 전가산기, 가산기, 증분기, 산술 논리 장치를 구현하세요.**

* 가산기
* 증분기
* 산술 논리 장치

> 제출

```plaintext
작성한 모든 *.hdl 파일을 project2.zip이라는 이름의 zip 파일로 압축하여(폴더 안에 넣지 말고 파일 자체만 압축) 제출해야 합니다. 코스에서 지정하지 않은 도우미 칩을 빌드하는 경우 해당 도우미 칩을 zip 파일에 포함해야 합니다. 제출 횟수는 무제한이며 성적은 모든 제출물의 최대치가 되므로 다시 제출해도 점수를 잃지 않습니다.

감사관으로 코스를 수강하는 경우 설명된 테스트를 사용하여 자신의 작업을 직접 확인할 수 있습니다.
여기
. 인증서 옵션을 수강하는 경우 여기에서 프로젝트 zip 파일을 제출하세요.
```

### Table of Contents

* [Adders](project2.md#adders)
* [Incrementer](project2.md#incrementer)
* [Arithmetic Logic Unit](project2.md#arithmetic-logic-unit)

### Adders

> Half Adder

* chip: HalfAdder
* input: a,b
* output: sum,carry
* a+b
  * carry: 올림 수
  * sum: 더한 결과 값

![image](<../../.gitbook/assets/project2_image0002 (1).png>)

*   implement

    ```
      CHIP HalfAdder {
          IN a, b;    // 1-bit inputs
          OUT sum,    // Right bit of a + b 
              carry;  // Left bit of a + b

          PARTS:
          Xor(a=a, b=b, out=sum);
          And(a=a, b=b, out=carry);
      }
    ```

> Full Adder

* chip: FullAdder
* input: a, b, c
* output: sum, carry
* a + b + c
  * sumTemp = a + b
  * sum = sumTemp + c
  * carry = carry

![image](<../../.gitbook/assets/project2_image0003 (1).png>)

*   implement

    ```
      CHIP FullAdder {
          IN a, b, c;  // 1-bit inputs
          OUT sum,     // Right bit of a + b + c
              carry;   // Left bit of a + b + c

          PARTS:
          HalfAdder(a=a, b=b, sum=temp, carry=temp2);
          HalfAdder(a=temp, b=c, sum=sum, carry=temp3);
          Xor(a=temp2, b=temp3, out=carry);
      }
    ```

> Adder 16-bit

* chip: Adder16
* input: a\[16], b\[16]
* output: out\[16]
* multi-bit Adder
*   implement

    ```
      CHIP Add16 {
          IN a[16], b[16];
          OUT out[16];

          PARTS:
          HalfAdder(a=a[0], b=b[0], sum=out[0], carry=tempCarry1);
          FullAdder(a=a[1], b=b[1], c=tempCarry1, sum=out[1], carry=tempCarry2);
          FullAdder(a=a[2], b=b[2], c=tempCarry2, sum=out[2], carry=tempCarry3);
          FullAdder(a=a[3], b=b[3], c=tempCarry3, sum=out[3], carry=tempCarry4);
          FullAdder(a=a[4], b=b[4], c=tempCarry4, sum=out[4], carry=tempCarry5);
          FullAdder(a=a[5], b=b[5], c=tempCarry5, sum=out[5], carry=tempCarry6);
          FullAdder(a=a[6], b=b[6], c=tempCarry6, sum=out[6], carry=tempCarry7);
          FullAdder(a=a[7], b=b[7], c=tempCarry7, sum=out[7], carry=tempCarry8);
          FullAdder(a=a[8], b=b[8], c=tempCarry8, sum=out[8], carry=tempCarry9);
          FullAdder(a=a[9], b=b[9], c=tempCarry9, sum=out[9], carry=tempCarry10);
          FullAdder(a=a[10], b=b[10], c=tempCarry10, sum=out[10], carry=tempCarry11);
          FullAdder(a=a[11], b=b[11], c=tempCarry11, sum=out[11], carry=tempCarry12);
          FullAdder(a=a[12], b=b[12], c=tempCarry12, sum=out[12], carry=tempCarry13);
          FullAdder(a=a[13], b=b[13], c=tempCarry13, sum=out[13], carry=tempCarry14);
          FullAdder(a=a[14], b=b[14], c=tempCarry14, sum=out[14], carry=tempCarry15);
          FullAdder(a=a[15], b=b[15], c=tempCarry15, sum=out[15], carry=carry);

      }
    ```

### Incrementer

Incrementer는 주어진 값에 +1 하는 기능을 맡는다. HalfAdder 에서 a+b에 b값으로 1을 주고, a를 원하는 값을 넣을 수도 있지만 이런 증분기가 있음으로 유용하게 쓰일 수 있기 때문에 기능 정의를 한다고 안내한다.

> Incrementer 16-bit

* chip: Inc16
* input: in\[16]
* output: out\[16]
*   implement

    ```
      CHIP Inc16 {
          IN in[16];
          OUT out[16];

          PARTS:
          Add16(a[0]=true, b=in, out=out);
      }
    ```

### Arithmetic Logic Unit

> ALU

![image](<../../.gitbook/assets/project2_image0004 (1).png>)

*   implement

    ```
      CHIP ALU {
          IN  
              x[16], y[16],  // 16-bit inputs        
              zx, // zero the x input?
              nx, // negate the x input?
              zy, // zero the y input?
              ny, // negate the y input?
              f,  // compute out = x + y (if 1) or x & y (if 0)
              no; // negate the out output?

          OUT 
              out[16], // 16-bit output
              zr, // 1 if (out == 0), 0 otherwise
              ng; // 1 if (out < 0),  0 otherwise

          PARTS:
          // ######################## zx & nx ###############################
          // ======================== comment ===============================
          // zx: 1 -> x is zero (set)
              // :: if zx has 1 = tempX is 0 because b is false.
          // nx: 1 -> transform x (not)
              // first, assignment not X value
              // :: if nx has 1 = call not(x) -> by Mux16
          // ======================== comment ===============================
          
          Mux16(a=x, b=false, sel=zx, out=tempX);  // here is zx
          Not16(in=tempX, out=notTX);
          Mux16(a=tempX, b=notTX, sel=nx, out=X);  // here is nx calling

          // ######################## zy & ny ###############################
          // ======================== comment ===============================
          // about y, same to X.
          // ======================== comment ===============================

          Mux16(a=y, b=false, sel=zy, out=tempY);
          Not16(in=tempY, out=notTY);
          Mux16(a=tempY, b=notTY, sel=ny, out=Y);
          // ========================== code ================================

          // ########################### f ##################################
          // ======================== comment ===============================
          // function: 1 -> Add, function: 0 -> And
              // :: Mux16 -> selector: 1 = run b, else run a.
              // :: function bit done.
          // ======================== comment ===============================

          Add16(a=X, b=Y, out=addXY);
          And16(a=X, b=Y, out=andXY);
          Mux16(a=andXY, b=addXY, sel=f, out=functionResult);
          // ========================== code ================================

          // ########################## output ###############################
          // ======================== comment ===============================
          // about no, same to function(f) -> output to "out[16]"
          // ng: multi-bit indexes are left end is negative
          // LSB, MSB: 16-bit out transform 1bit zr
              // by Or8way -> input 8bit, output 1bit
              // 16bit / 2 -> 8bit *2 = Calling "Or8Way" twice can convert to "1bit out"
              // first Or8way -> LSB
              // second Or8way -> MSB
          // ======================== comment ===============================

          Not16(in=functionResult, out=notFunctionResult);
          Mux16(a=functionResult, b=notFunctionResult, sel=no, out=out, out[0..7]=right, out[8..15]=left, out[15]=ng);


          // ======================== comment ===============================
          // Or -> 16-bit result to 1-bit result
              // Not -> linkedOut has 0 -> zr is 1
              // linkedOut has 1 -> zr is 0
          // ======================== comment ===============================
          Or8Way(in=right, out=right8BitOut);  // 0
          Or8Way(in=left, out=left8BitOut);  // 
          Or(a=right8BitOut, b=left8BitOut, out=linkedOut);  // 1
          
          Not(in=linkedOut, out=zr);  // 0
          // ========================== code ================================
      }
    ```

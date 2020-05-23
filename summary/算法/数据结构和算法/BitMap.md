# BitMap

## java.util包中的实现BitSet

待补充

## BitMap

```java
class BitMap {
    private int[] bitsMap;
    public BitMap(long length){
        // 因为int是32位的，右移5位相当于除以32，我们需要的是求出每个数应该在数组中的第i个位置以及该
        // 位置上的偏移。若不是基于int，如基于byte，则将>>5改成>>3，&31改成&7
        bitsMap = new int[(int)(length >> 5) + ((length & 31) > 0 ? 1 : 0)]; // 余数大于0需要再加一位来放余数
        /**另外，学到了只要是2的指数次，与该数求余都可以用 (x & (2^n - 1))。
         * 因为，与2的指数次求余，相当于每次减2的指数次，那么求余的结果就是上述的结果
         */
        System.out.println(bitsMap.length + " " + (int)(length >> 5) + " " + (int)(length & 31));
    }
    /**
     * setBit的流程为：1. 求出num的位置idx
     * 2. 求出offset，用 | 将idx的offset位置置为1
     * @param bitIndex 将被映射的数
     */
    public void setBit(long bitIndex){
        int idx = (int)(bitIndex >> 5);
        // int offset = (num & 31);
        bitsMap[idx] |= 1 << (bitIndex & 31);
    }
    /**
     * getBit的流程为：1. 求出num的位置idx的值data
     * 2. 求出offset，将data右移offset后&1，即可得到该数是否存在
     * @param bitIndex
     * @return 返回该数是否存在，存在则在位置idx的offset处为1，否则为0；或者true or false
     */
    public boolean getBit(long bitIndex){
        int data = bitsMap[(int)(bitIndex >> 5)];
        int offset = (int)(bitIndex & 31);
        return (data >> offset & 1) == 1;
        // return intData >> offset & 1; 
    }
}
```


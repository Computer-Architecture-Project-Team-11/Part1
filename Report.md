# Advanced Computer Architecture Project -Part 1

1. Based on the contents of the `starter_se.py` file, assumptions regarding the characteristics that have been passed to gem5 after running the `HelloWorld` program can be made. More specifically:
   * **CPU Type = "atomic"** (the type of the cpu is defined in line 185:
      ```python
      parser.add_argument("--cpu", type=str, choices=list(cpu_types.keys()),
                        default="atomic"
      ```  
      The final value though is determined by the argument in the terminal command `--cpu="minor"`. This means that the cache classes are the ones defined in `starter_se.py` for the "minor" type.
   * **Operational Frequency = 1GHz** :
      ```python
      parser.add_argument("--cpu-freq", type=str, default="1GHz")
      ```  
   * **Number of Cores = 1**:
      ```python
        parser.add_argument("--num-cores", type=int, default=1,
                        help="Number of CPU cores")
       ```
   * **Memory Type = DDR3_1600_8x8**:
      ```python
      parser.add_argument("--mem-type", default="DDR3_1600_8x8",
                        choices=ObjectList.mem_list.get_names(),
                          help = "type of memory to use")
        ```  
   * **Number of Memory Channels = 2**:
       ```python
      parser.add_argument("--mem-channels", type=int, default=2,
                          help = "number of memory channels")
        ```  
   * **Number of Memory Ranks per Channel = 0**:
       ```python
      parser.add_argument("--mem-ranks", type=int, default=None,
                        help = "number of memory ranks per channel")
        ```  
   * **Memory Size = 2GB**:
       ```python
     parser.add_argument("--mem-size", action="store", type=str,
                        default="2GB",
                        help="Specify the physical memory size")
        ```  
2. The following are information dirived from the `stats.txt`, `config.ini` and `config.json` files after running this command:

        
           $ ./build/ARM/gem5.opt -d hello_result configs/example/arm/starter_se.py --cpu="minor" "tests/test-progs/hello/bin/arm/linux/hello"
        

   a. 
      * Based on the `config.ini` file the cpu-type is "minor"
          ```
             [system.cpu_cluster.cpus]
             type=MinorCPU
          ```
         Although as mentioned in part 1. the default value, defined in `starter_se.py`, for the cpu-type is "atomic", the final value is determined by the argument in the above command `--cpu="minor"`.
      * from `statistics.txt` the operational frequency is 1GHZ as expected
         ```
          sim_freq                                 1000000000000                       # Frequency of simulated ticks
         ```
      * memory type
      * number of memory channels
      * from `config.ini` the number of memory ranks per channel is 2
         ```
          [system.mem_ctrls0]/[system.mem_ctrls1]
          ranks_per_channel=2
         ```
      * memory size

   b. 
      * sim_seconds: the time in seconds that the simulation needs for execution
      * sim_insts: the number of instructions that were simulated
      * host_inst_rate: the rate of instructions simulated per second

   c.  ```
       system.cpu_cluster.cpus.committedInsts           5027                       # Number of instructions committed
       ```

   d.  from `starter_se.py` and because the type of the CPU is "minor" the L2 Cache class is "devices.L2".
       The number of times that the L2 cache was accessed is equal to the muner of times L1 was accessed and a miss occured which is 147.
       ```
        system.cpu_cluster.cpus.branchPred.indirectMisses          147                       # Number of indirect misses.
        ```

3. Different in-order CPU models on gem5:
   
     i. SimpleCPU:    
        * BaseSimpleCPU:  
        * AtomicSimpleCPU:
     
     >This version of SimpleCPU uses atomic memory accesses. It derives from BaseSimpleCPU and implements functions to read and write memory, as well as to tick,            meaning what happens in every CPU cycle. Also, it estimates the overall cache access time using the latency estimates from the atomic accesses. Lastly, the            AtomicSimpleCPU defines the port that is used to hook up to memory, and connects the CPU to the cache.  
        * TimingSimpleCPU:
      
    ii. Minor CPU:
    >This in-order processor model has a fixed pipeline, but its data structures and execute behaviour are configurable. It provides a framework to match micro-            architecturally the model with similar processor which has strict in-order execution behaviour and visualises the position of an instruction in the pipeline            through the MinorTrace/minorview.py format/too. 

   a. Commands for the `fibonacci.c` file:

      * MinorCPU
         ``` 
            $ ./build/ARM/gem5.opt -d fib_results_MinorCPU configs/example/se.py --cpu-type=MinorCPU --caches -c tests/test-progs/hello/bin/arm/linux/fibonacci
          ```
      * TimingSimpleCPU
        ```
           $ ./build/ARM/gem5.opt -d fib_results_TimingSimpleCPU configs/example/se.py --cpu-type=TimingSimpleCPU --caches -c tests/test-progs/hello/bin/arm/linux/fibonacci
         ```

   b. srhttjdtn

   c. srthdhrg



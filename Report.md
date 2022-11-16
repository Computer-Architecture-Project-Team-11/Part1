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
      * from `config.ini` we can see that there are 2 memory channels:
         ```
         memories=system.mem_ctrls0 system.mem_ctrls1
         ```
      * from `config.ini` the number of memory ranks per channel is 2 for both memory channels
         ```
          [system.mem_ctrls0]/[system.mem_ctrls1]
          ranks_per_channel=2
         ```
      * memory size

   b. 
      * sim_seconds: the time in seconds that the simulation needs for execution
      * sim_insts: the number of instructions that were simulated
      * host_inst_rate: the rate of instructions simulated per second

   c. From `statistics.txt` the number of committed instructions is 5027 and of committed operations is 5831.

       
       system.cpu_cluster.cpus.committedInsts           5027                       # Number of instructions committed

       system.cpu_cluster.cpus.committedOps             5831                       # Number of ops (including micro ops) committed

     
   The reason that they are not equal is that "committedInsts" is the architectural number of assembly instructions executed while "commmittedOps" is the number of micro-operations. Each instruction can expand to multiple microoperations, so this number is always greater or equal than committedInsts.

    The number of instructions simulated is again 5027:
    
        sim_insts                                        5027                       # Number of instructions simulated

    The fact that the simulated instructions and committed instruction are the same signifies that all instructions are committed. An instruction commits only if it and all instructions before it have completed successfully.
       

   d.  from `starter_se.py` and because the type of the CPU is "minor" the L2 Cache class is "devices.L2".
       From `statistics.tct` the L2 accesses are the following:

       
       system.cpu_cluster.l2.overall_accesses::.cpu_cluster.cpus.inst          327                       # number of overall (read+write) accesses

       system.cpu_cluster.l2.overall_accesses::.cpu_cluster.cpus.data          147                       # number of overall (read+write) accesses

       system.cpu_cluster.l2.overall_accesses::total          474                       # number of overall (read+write) accesses
        

      The number of times that the L2 cache was accessed is equal to the muner of times L1 was accessed and a miss occured which is 147 (as far as data is concerned):

       
        system.cpu_cluster.cpus.branchPred.indirectMisses          147                       # Number of indirect misses.
        

3. Different in-order CPU models on gem5:
   
     i. SimpleCPU:  
      * BaseSimpleCPU: 

    > This version of SimpleCPU is a single cycle CPU which means it can execute only one instruction in a cycle. It can be inheritedd by AtomicSimpleCPU and TimingSimpleCPU. It can not be run on its own. One of the inheriting classes, either AtomicSimpleCPU or TimingSimpleCPU must be used. BaseSimpleCPU defines functions for checking for interrupts, setting up a fetch request, handling pre-execute setup, handling post-execute actions, and advancing the PC to the next instruction and holds architected state, stats common across the SimpleCPU models. It is also responsible for the ExecContext interface which reovides information regarding the status of the execution.
 
      * AtomicSimpleCPU:
     
     >This version of SimpleCPU uses atomic memory accesses. It derives from BaseSimpleCPU and implements functions to read and write memory, as well as to tick,            meaning what happens in every CPU cycle. Also, it estimates the overall cache access time using the latency estimates from the atomic accesses. Lastly, the            AtomicSimpleCPU defines the port that is used to hook up to memory, and connects the CPU to the cache.  
       
      * TimingSimpleCPU:

    >This version of SimpleCPU uses timing memory accesses. Timing accesses are the most detailed access but they lack in speed (delay). With the sending of a request function at some time, a response function or a number of functions are scheduled at some time in the future to be executed. It derives from BaseSimpleCPU and and implements the same set of functions as AtomicSimpleCPU. It defines the port that is used to hook up to memory, and connects the CPU to the cache. It also defines the necessary functions for handling the response from memory to the accesses sent out. If the response is a NACK the procedure gets repeated.
      
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


Resources:
* [Committed instructions differencies](https://stackoverflow.com/questions/65010636/difference-between-committed-instructions-and-committed-ops)
* [Committed instructions](https://my.eng.utah.edu/~cs6810/pres/12-6810-09.pdf)
* [CPU Models](https://www.gem5.org/documentation/general_docs/cpu_models/SimpleCPU#basesimplecpu)
* 



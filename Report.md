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
      * from `config.ini` we can see that there are 2 memory channels:
         ```
         memories=system.mem_ctrls0 system.mem_ctrls1
         ```
         The type for both channels is:
          ```
          type=DRAMCtrl
         ```
      
      * from `config.ini` the number of memory ranks per channel is 2 for both memory channels
         ```
          [system.mem_ctrls0]/[system.mem_ctrls1]
          ranks_per_channel=2
         ```

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
       From `statistics.txt` the L2 accesses are the following:

       
       system.cpu_cluster.l2.overall_accesses::.cpu_cluster.cpus.inst          327                       # number of overall (read+write) accesses

       system.cpu_cluster.l2.overall_accesses::.cpu_cluster.cpus.data          147                       # number of overall (read+write) accesses

       system.cpu_cluster.l2.overall_accesses::total          474                       # number of overall (read+write) accesses
        

      The number of times that the L2 cache was accessed is equal to the muner of times L1 was accessed and a miss occured which is 147 (as far as data is concerned):

       
        system.cpu_cluster.cpus.branchPred.indirectMisses          147                       # Number of indirect misses.
        

3. Different in-order CPU models on gem5:
   
     i. SimpleCPU:  
      * BaseSimpleCPU: 

    > This version of SimpleCPU is a single cycle CPU which means it can execute only one instruction in a cycle. It can be inheritedd by AtomicSimpleCPU and TimingSimpleCPU. It can not be run on its own. One of the inheriting classes, either AtomicSimpleCPU or TimingSimpleCPU must be used. BaseSimpleCPU defines functions, common for SimpleCPU models, for checking for interupts, controling set ups and actions and advancing the PC into the next instuction, holding the architected state. It is also responsible for the ExecContext interface which is used by the instructions to access the CPU.
 
      * AtomicSimpleCPU:
     
     >In this version of SimpleCPU atomic memory accesses are used. It derives from BaseSimpleCPU and uses functions to read and write memory, as well as to tick,  meaning what happens in every CPU cycle. Also, it determines the overall cache access time using the latency estimates from the atomic accesses. Lastly, the AtomicSimpleCPU defines which port is used to connect to memory, and attaches the CPU to the cache.  
       
      * TimingSimpleCPU:

    >This version of SimpleCPU uses timing memory accesses. Timing accesses are the most detailed access but they lack in speed (delay). With the sending of a request function at some time, a response function or a number of functions are scheduled at some time in the future to be executed. It derives from BaseSimpleCPU and and implements the same set of functions as the aforementioned model, AtomicSimpleCPU. It is responsible for the handling of the response from the memory to the accesses sent out.  If the response is a NACK the procedure gets repeated. It, finally, defines the port that is used to hook up to memory, and connects the CPU to the cache.
      
    ii. Minor CPU:
    >This in-order processor model has a standar pipeline, but its data structures and execute behaviour are configurable. It provides a framework to match the model with a similar processor on a micro-architectural level which has strict in-order execution behaviour and visualises the position of an instruction in the pipeline using the MinorTrace/minorview.py format/too. 

   a. Commands for the `fibonacci.c` file:

      * MinorCPU
         ``` 
            $ ./build/ARM/gem5.opt -d fib_results_MinorCPU configs/example/se.py --cpu-type=MinorCPU --caches -c tests/test-progs/hello/bin/arm/linux/fibonacci
          ```
      * TimingSimpleCPU
        ```
           $ ./build/ARM/gem5.opt -d fib_results_TimingSimpleCPU configs/example/se.py --cpu-type=TimingSimpleCPU --caches -c tests/test-progs/hello/bin/arm/linux/fibonacci
         ```

      Execution times:
      * MinorCPU
        ```
        sim_seconds                                  0.000333                       # Number of seconds simulated
        ```
      * TimingSimpleCPU
         ```
        sim_seconds                                  0.000692                       # Number of seconds simulated
        ```

   b. Comparing the simulations run with MinorCPU and TimingSimpleCPU we noticed the following:
     * Differencies:  
       * host_seconds: 0.78 for TimingSimpleCPU and 2,09 for MinorCPU  
       * sim_seconds: 0.000692 for TimingSimpleCPU and 0,000333 for MinorCPU  
       * Number of indirect misses: 138 for MinorCPU and it is not mentioned for TimingSimpleCPU  
     * Similarities:
       * sim_freq: 1000000000000 for both CPUs
       * host_mem_usage: 674144 for TimingSimpleCPU and 678752 for MinorCPU
       * Number of instructions commited: 445920 for TimingSimpleCPU and 445973 for MinorCPU  
    
    The differencies are mostly related to time. They are justified considering the fact that TimingSimpleCPU lacks in speed in comparison to MinorCPU.  
    The similarities are also justified because they concern a) the frequency which is determined by us, b) the memory used by the host and the number of instructions commited which depend on our program and are not influenced by the CPU.

   c. Changed parameters and comparison of the 2 CPU models:
   
      * Different memory type:
        * For the TimingSimpleCPU we used the "LPDDR2_S4_1066_1x32" memory type:
          ```
          $ ./build/ARM/gem5.opt -d fib_results_TimingSimpleCPULPDDR2_S4_1066_1x32 configs/example/se.py --cpu-type=MinorCPU --mem-type=LPDDR2_S4_1066_1x32 --caches -c tests/test-progs/hello/bin/arm/linux/fibonacci
          ```
          * There is a small increment in the time of execution for while using the "LPDDR2_S4_1066_1x32" memory type. Number of seconds simulated before = 0.000692 and after = 0.000700. 
          * Also a decrement in the instruction rate is noticed: Simulator instruction rate before=574316  and after=561871. 
        * For the MinorCPU we used the "SimpleMemory" memory type:
          ```
           $ ./build/ARM/gem5.opt -d fib_results_MinorCPUSimpleMemory configs/example/se.py --cpu-type=MinorCPU --mem-type=SimpleMemory --caches -c tests/test-progs/hello/bin/arm/linux/fibonacci
          ```
           * The instruction rate has decreased a lot in with the use of the "SimpleMemory" type. Specifically, simulator instruction rate before=213505  and after=151692. (respective decrement for the ops rate)
           * The simulated time seems to be slighttly decreased. Number of seconds simulated before = 0.000333 and after = 0.000326. 
           * Finally this memory type seems to have an small affect on the indirect misses since they are increased by 1.

      * Different Operational Frequency:
         * For the TimingSimpleCPU we set the frequency to 4GHz:
            ```
            $ ./build/ARM/gem5.opt -d fib_results_TimingSimpleCPU4GHz configs/example/se.py --cpu-type=TimingSimpleCPU --cpu-clock=4GHz --caches -c tests/test-progs/hello/bin/arm/linux/fibonacci
             ```
         * For the MinorCPU we set the frequency to 4GHz:
            ```
            $ ../build/ARM/gem5.opt -d fib_results_MinorCPU4GHz configs/example/se.py --cpu-type=MinorCPU --cpu-clock=4GHz --caches -c tests/test-progs/hello/bin/arm/linux/fibonacci
             ```
          The operational frequency refers to the processor's operational clock cycles per second. This means that by increasing the frequency the execution must be faster. We can confirm that by the results on the `statistics.txt` file. More specifically we observed subduplication of the total simulated time in both CPU models with the quadruplication of the operational frequency.
        
Resources:
* [Committed instructions differencies](https://stackoverflow.com/questions/65010636/difference-between-committed-instructions-and-committed-ops)
* [Committed instructions](https://my.eng.utah.edu/~cs6810/pres/12-6810-09.pdf)
* [CPU Models](https://www.gem5.org/documentation/general_docs/cpu_models/SimpleCPU#basesimplecpu)


## Overall review:

We found that this lab exercise was educating on many levels:
* Firstly, we got acquainted with GitHub and Markdown language, which seems to be a usefull skill to possess for our future projects and a great way to organize our work.  
* Also, we were able to understand the effect of using different CPU models and changing different parameters can have on the execution of a program, mainly on the simulation time. 
* First contact with gem5. We fimiliarized ourselves with its basic features and possibilities, observed the things happening in the back-ground will executing a program and how the whole procedure can be affected by changing different parameters like the CPU models or the operational frequency.
* We also had the chance to see in action a lot of procedures and results we had discussed in the lectures such as the L1 and L2 relation regarding the misses of the first and the accesses to the second.
* The only difficulty we faced was finding the appropriate argument to add to the command in terminal in order to change a specfic parameter. It whould be very helpful if the parameters and their respective arguments where all gathered in one place.
* Finally, the assignment pdf was really detailed and instructive. 




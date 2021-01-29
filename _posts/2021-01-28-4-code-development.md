---
title: "Code Development"
description: 30
---

This section provides instructions for calculating BMR (*Basal Metabolic Rate)* with the Accelerate Kit API using **Mifflin-St Jeor Equation:**

For men:     BMR = (10* weight [kg]) + (6.25* Height [cm]) – (5* age [years]) + 5

For women:   BMR = (10* weight [kg]) + (6.25* Height [cm]) – (5* age [years]) - 161

In this code BMR is calculated for 1000000 people with their weight, size and age arranged randomly between certain ranges.

<aside class="special">
	<p><strong>Note:</strong> In this case, the calculation of BMR is calculated for 1000000 people. You can select the total number of people that is more suitable for your computer based on your actual situation.</p>
</aside>

<aside class="special">
	<p><strong>Note:</strong> The higher the number of people used in the <Strong>BMR</Strong> calculation means the more obvious the difference in calculation between Java, single tread and multi treads.</p>
</aside>

1. Set related parameters in the Acceleration Kit native-lib.cpp class, including the **sumOfBMR**(Result BMR), **counterHowManyPeople** and **threads**(Tread number)

```cpp
__block double sumOfBMR = 0;
    __block double counterHowManyPeople = 0;

    std::ifstream file("/data/user/0/com.huawei.acceleratekitcodelab/files/data.txt");
    std::string str;

     string *dataArray = new string[totalNumberOfData];
    int counter=0;
    while(std::getline(file,str))
    {
        dataArray[counter++] = str;
    }

    /***
         * Male
         * (10 x weight)+(6.25 + height)-(5 x age)+5
         *
         * Female
         * (10 x weight)+(6.25 + height)-(5 x age)-161
         *
         * Name,gender,age,weight,height;
         */
    timestamp_t t0 = get_timestamp();

    int eachThreadCalculate = totalNumberOfData/100;

    int threads = totalNumberOfData/eachThreadCalculate; // thread count is 100, 100 thread will run

```

 

2. Call **dispatch_queue_create** to create a serial queue for **BMR** calculation. Then, call **dispatch_apply** to execute a task for multiple times in a specified queue synchronously. In general, you are advised to set the specified queue to **DISPATCH_APPLY_AUTO**, which is a global concurrent queue whose priority is closest to that of the current queue. Since tasks are executed concurrently, and the **BMR** calculation in this example is the same, it is possible to accelerate the calculation in a manner of concurrent execution.

```cpp
dispatch_apply(threads, DISPATCH_APPLY_AUTO, ^(size_t idx){
            double currentBRM = 0.0;
            int currentCounterHowManyPeople = 0;

        for(int i=idx*eachThreadCalculate;i<(idx*eachThreadCalculate)+eachThreadCalculate;i++)
        {
            int *currentData = getValueInString(dataArray[i]).array;

            if(currentData[1]==0)//male
            {
                currentBRM += (((10*currentData[3])+(6.25*currentData[4])
                               -(5*currentData[2]))+5);


                currentCounterHowManyPeople++;
            }
            else if(currentData[1]==1)//female
            {
                currentBRM += (((10*currentData[3])+(6.25*currentData[4])
                                -(5*currentData[2]))-161);
                currentCounterHowManyPeople++;
            }
        }

        dispatch_sync(accumulator, reinterpret_cast<dispatch_block_t>(^{
          counterHowManyPeople +=currentCounterHowManyPeople;
            sumOfBMR += currentBRM;
        }));
    }
```

3. Implement the calculation algorithm of **BMR** for men and women by using different formulas. That is, allocate the number of people/1000 terms into threads tasks (threads=totalNumberOfData/1000). The start and end of each task are calculated based on the data of sample people

   ```cpp
   for(int i=idx*eachThreadCalculate;i<(idx*eachThreadCalculate)+eachThreadCalculate;i++)
           {
               int *currentData = getValueInString(dataArray[i]).array;
   
               if(currentData[1]==0)//male
               {
                   currentBRM += (((10*currentData[3])+(6.25*currentData[4])
                                  -(5*currentData[2]))+5);
   
   
                   currentCounterHowManyPeople++;
               }
               else if(currentData[1]==1)//female
               {
                   currentBRM += (((10*currentData[3])+(6.25*currentData[4])
                                   -(5*currentData[2]))-161);
                   currentCounterHowManyPeople++;
               }
           }
   ```

   4. Call **dispatch_sync** to calculate the sum of each task for **BMR** and counter for that how many times is **BMR** calculated.

      ```cpp
      dispatch_sync(accumulator, reinterpret_cast<dispatch_block_t>(^{
                counterHowManyPeople +=currentCounterHowManyPeople;
                  sumOfBMR += currentBRM;
              }));
      ```

      5. Call **dispatch_release** to release the space of the accumulator. Objects created by APIs of class **dispatch_xxx_create()** each correspond to a space on the heap. Because there is no garbage collection mechanism, such objects created by the user need to be proactively released when the user determines that the objects are no longer needed.

      ```cpp
      dispatch_release(accumulator);
      ```

      6. Output the accumulated string for BMR to the screen for display.

      ```cpp
      extern "C" JNIEXPORT jstring JNICALL
      Java_com_huawei_acceleratekitcodelab_MainActivity_calculateAverageBRMs(
              JNIEnv* env,
              jobject,
              jint how_many_people)
      {
          std::stringstream ss;
          ss << std::setprecision(0) << calculateAverageBRMs(how_many_people);
          return env->NewStringUTF(ss.str().c_str());
      }
      ```

      7. Create the **calculateAverageBRMs** to calculate **Average BMR** with from Java Class like below**.**

      ```cpp
      public native String calculateAverageBRMs(int howManyPeople);
      ```

      8. In java class, call **calculateAverageBRMs** to calculate **Average BMR** with **Accelerate Kit**.

      ```java
      public void startCalculation()
          {
              sumOfBMR = 0.0;
              counterHowManyPeople = 0;
              StringBuffer buffer = new StringBuffer();
      
              buffer.append("\n Average BRM's of "+peopleList.size()+" people.\n\n");
      
              // Accelerate Kit
              buffer.append("  Accelerate Kit: ");
              String[] accelerateKitResult = new String[4];
              accelerateKitResult = calculateAverageBRMs(peopleList.size()).split(",");
      
              buffer.append("\nAccelerate Kit \nAverage BMR: "+Double.parseDouble(accelerateKitResult[0])/Double.parseDouble(accelerateKitResult[1])
                      +"\nHow Many People: "+Double.parseDouble(accelerateKitResult[1])+ "\ntime: " + accelerateKitResult[2] + "milliseconds.\n ");
      
          }
      
      /**
           * A native method that is implemented by the 'native-lib' native library,
           * which is packaged with this application.
           */
          public native String calculateAverageBRMs(int howManyPeople);
      ```

      
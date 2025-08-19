# Task8

The purpose of this task is to rewrite the previously written **vector_sum** code using a different approach. In C, the vector sum can be represented with a FOR loop that goes through all the components of the vectors, summing them one by one:
```
for (int i = 0; i< N-1; i++) {
  d[i] = a * x[i] + y[i];
}
```
Now, we will divide this operation into chunks: we take intervals of vectors by splitting the $N$ components into $N_{chunks}$ groups. At this point, we run a for loop for each group, and then combine everything together to get the final result.

In detail, for this exercise, we take $N_{chunks}=ceil(N/8)$, where ``ceil`` returns the nearest integer number. Finally, we test the code by comparing the total sum of all components with the combined sum of the individual chunks.

## Code in Julia
```
function daxpyJ_chunks(N, a, xval, yval)
    x = fill(xval, N)
    y = fill(yval, N)
    d = zeros(N)

#here we define the chunks
    chunk_size = 8
    number_of_chunks = ceil(Int, N / chunk_size)
    partial_sums = zeros(number_of_chunks)

    for chunk_index in 0:number_of_chunks - 1
        #first and last componenet of each chunks
        current_start = chunk_index * chunk_size + 1
        current_end = min(current_start + chunk_size - 1, N)
        #local sum
        for i in current_start:current_end
            d[i] = a * x[i] + y[i]
            partial_sums[chunk_index + 1] += d[i]
        end
    end

    #println("Partial sums: ", partial_sums)
    println("Total sum from chunks: ", sum(partial_sums))
    println("Direct sum: ", sum(d))
    println("First 10 components of d=a*x+y", d[1:min(10, N)])
end
```
To run the code, save it (for example sum_chunks.jl) and run it direclty from the bash:
```
julia sum_chunks.jl 10 2 3 4
```
where in this examples the imput is (N,a,x,y).

## Code in C
The code in C is quite similar with respect to the Julia's one.

```#include <stdio.h>
#include <stdlib.h>
// vectors initialization
void daxpyC_chunks(int N, double a, double xval, double yval) {
    double *x = (double *)malloc(N * sizeof(double));
    double *y = (double *)malloc(N * sizeof(double));
    double *d = (double *)malloc(N * sizeof(double));
    
    int chunk_size = 8;
    int number_of_chunks = (N + chunk_size - 1) / chunk_size;
    //or    int number_of_chunks = (int) ceil((double) N / chunk_size); by addying #include <math.h>
    double *partial_sums = (double *)calloc(number_of_chunks, sizeof(double));
    
    for (int i = 0; i < N; i++) {
        x[i] = xval;
        y[i] = yval;
    }

    
    for (int chunk_index = 0; chunk_index < number_of_chunks; chunk_index++) {
        int current_start = chunk_index * chunk_size;
        int current_end = current_start + chunk_size;
        if (current_end > N) current_end = N;
        //local sum
        for (int i = current_start; i < current_end; i++) {
            d[i] = a * x[i] + y[i];
            partial_sums[chunk_index] += d[i];
        }
    }
    //local sum check
    double total_sum = 0.0;
    for (int i = 0; i < number_of_chunks; i++) {
        total_sum += partial_sums[i];
    }
    
    double direct_sum = 0.0;
    for (int i = 0; i < N; i++) {
        direct_sum += d[i];
    }
    
    printf("Total sum from chunks: %f\n", total_sum);
    printf("Direct sum: %f\n", direct_sum);
    printf("First 10 components of d = a * x + y:\n");
    int print_count = (N < 10) ? N : 10;
    for (int i = 0; i < print_count; i++) {
        printf("d[%d] = %f\n", i, d[i]);
    }
    
    free(x);
    free(y);
    free(d);
    free(partial_sums);
}
//call the function
int main(int argc, char *argv[]) {
    if (argc != 5) {
        printf("Usage: %s N a xval yval\n", argv[0]);
        return 1;
    }
    
    int N = atoi(argv[1]);
    double a = atof(argv[2]);
    double xval = atof(argv[3]);
    double yval = atof(argv[4]);
    
    daxpyC_chunks(N, a, xval, yval);
    
    return 0;
}

```

In this code, to split the operation into chunks, we can use either `ceil()` or `(N + chunk_size - 1) / chunk_size` by including `math.h`; both approaches are equivalent in C.
To use the last code, simply save it, compile it, and run it by passing N, a, x, and y as arguments.

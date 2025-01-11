# Dynamic-Programming-and-19-keys-

import java.io.File;
import java.io.FileNotFoundException;
import java.util.Scanner;

class Prog4Main {

   public static void main(String[] args) {
      double[] p = new double[10];
      double[] j = new double[6];
      double f;

      try {
         // Create a Scanner to read from the file
         Scanner scanner = new Scanner(new File("p4test1.txt"));

         // Read values for array p
         for (int i = 0; i < 10; i++) {
             if (scanner.hasNextDouble()) {
                 p[i] = scanner.nextDouble();
             }
         }

         // Read values for array j
         for (int i = 0; i < 6; i++) {
             if (scanner.hasNextDouble()) {
                 j[i] = scanner.nextDouble();
             }
         }

         // Read the value of f
         if (scanner.hasNextDouble()) {
             f = scanner.nextDouble();
         } else {
             throw new IllegalArgumentException("File does not contain enough data for f");
         }

         // Initialize the return objects
         Prog4Return p4return = new Prog4Return();
         p4return.action[0][0] = 1;
         p4return = Prog4.Prog4(p, j, f);

         // Print base case results
         System.out.println("Base case :");
         for (int i = 0; i <= 10; i++) {
             System.out.print(i + " : ");
             for (int k = 0; k <= 9; k++) {
                 System.out.print("(" + p4return.expected[i][k] + " , " + p4return.action[i][k] + ") ");
             }
             System.out.println(" ");
         }

         Prog4ReturnE p4returne = new Prog4ReturnE();
         p4returne.action[0][0][0] = 1;
         p4returne = Prog4.Prog4E(p, j, f);

         // Print extra credit results
         System.out.println("\nExtra Credit :\n");
         
         System.out.println("\nNo lifeline :\n");
         for (int i = 0; i <= 10; i++) {
             System.out.print(i + " : ");
             for (int k = 0; k <= 9; k++) {
                 System.out.print("(" + p4returne.expected[i][k][0] + " , " + p4returne.action[i][k][0] + ") ");
             }
             System.out.println(" ");
         }

         System.out.println("\nWith lifeline :\n");
         for (int i = 0; i <= 10; i++) {
             System.out.print(i + " : ");
             for (int k = 0; k <= 9; k++) {
                 System.out.print("(" + p4returne.expected[i][k][1] + " , " + p4returne.action[i][k][1] + ") ");
             }
             System.out.println(" ");
         }

         scanner.close(); // Close the scanner
      } catch (FileNotFoundException e) {
         System.out.println("File not found: " + e.getMessage());
      } catch (IllegalArgumentException e) {
         System.out.println("Invalid data: " + e.getMessage());
      }
   }
}

//testing file
0.1 0.2 0.3 0.4 0.5 0.6 0.7 0.8 0.9 0.95
10 20 40 60 80 120
0.75

public class Prog4Return {

	double expected[][] = new double [11][10];
	int action[][] = new int[11][10];

}


public class Prog4 {

    public static Prog4Return Prog4(double[] p, double[] j, double f) {
        Prog4Return result = new Prog4Return();

        // Initialize the DP arrays
        double[][] expected = new double[11][10];
        int[][] action = new int[11][10];

        // Pre-calculate jackpot values
        double[] jackpot = new double[11];
        for (int i = 1; i <= 6; i++) {
            jackpot[i] = j[i - 1];
        }
        for (int i = 7; i < 11; i++) {
            jackpot[i] = jackpot[6] * Math.pow(f, i - 6);
        }

        // Base cases
        for (int k = 1; k <= 9; k++) {
            expected[10][k] = jackpot[10] / k;
            action[10][k] = 1; // Must pick a key at the last step
        }

        // Fill DP table for q = 9 to 0
        for (int q = 9; q >= 0; q--) {
            for (int k = 0; k <= 9; k++) {
                if (q == 0 || k == 0) {
                    expected[q][k] = 0.0;
                    action[q][k] = 0;
                } else {
                    double pickKey = jackpot[q] / k;
                    double answerCorrect = (k > 1) ? expected[q + 1][k - 1] : expected[q + 1][1];
                    double answerIncorrect = expected[q + 1][Math.min(k + 1, 9)];
                    double answerNext = p[q] * answerCorrect + (1 - p[q]) * answerIncorrect;

                    if (pickKey >= answerNext) {
                        expected[q][k] = pickKey;
                        action[q][k] = 1;
                    } else {
                        expected[q][k] = answerNext;
                        action[q][k] = 0;
                    }
                }
            }
        }

        result.expected = expected;
        result.action = action;

        return result;
    }

public static Prog4ReturnE Prog4E(double[] p, double[] j, double f) {
    Prog4ReturnE result = new Prog4ReturnE();

    // Initialize the 3D DP arrays
    double[][][] expected = new double[11][10][2];
    int[][][] action = new int[11][10][2];

    // Pre-calculate jackpot values
    double[] jackpot = new double[11];
    for (int i = 1; i <= 6; i++) {
        jackpot[i] = j[i - 1];
    }
    for (int i = 7; i < 11; i++) {
        jackpot[i] = jackpot[6] * Math.pow(f, i - 6);
    }

    // Base cases for q = 10
    for (int k = 1; k <= 9; k++) {
        for (int u = 0; u <= 1; u++) {
            expected[10][k][u] = jackpot[10] / k; // Always pick a key at q = 10
            action[10][k][u] = 1; // Pick key
        }
    }

    // Fill DP table from q = 9 to 0
    for (int q = 9; q >= 0; q--) {
        for (int k = 0; k <= 9; k++) {
            for (int u = 0; u <= 1; u++) {
                if (q == 0 || k == 0) {
                    expected[q][k][u] = 0.0;
                    action[q][k][u] = 0; // No action possible
                } else {
                    // Calculate expected values for actions
                    double pickKey = jackpot[q] / k;

                    double answerCorrect = (k > 1) ? expected[q + 1][k - 1][u] : expected[q + 1][1][u];
                    double answerIncorrect = expected[q + 1][Math.min(k + 1, 9)][u];
                    double answerNext = p[q] * answerCorrect + (1 - p[q]) * answerIncorrect;

                    double useLifeline = 0.0;
                    if (u == 1 && q < 10) { // Lifeline can only be used before q = 10
                        double success = jackpot[q]; // Winning jackpot with the correct key
                        double failure = ((k - 1.0) / k) * expected[q + 1][Math.min(k + 1, 9)][0]; // Probability of failure
                        useLifeline = (1.0 / k) * success + failure; // Weighted expectation
                    }

                    // Decide the optimal action
                    if (useLifeline > pickKey && useLifeline > answerNext && u == 1) {
                        expected[q][k][u] = useLifeline;
                        action[q][k][u] = 2; // Use lifeline
                    } else if (pickKey >= answerNext) {
                        expected[q][k][u] = pickKey;
                        action[q][k][u] = 1; // Pick key
                    } else {
                        expected[q][k][u] = answerNext;
                        action[q][k][u] = 0; // Answer the next question
                    }
                }
            }
        }
    }

    // Store results in Prog4ReturnE object
    result.expected = expected;
    result.action = action;

    return result;
}

}

public class Prog4ReturnE {

	double expected[][][] = new double [11][10][2];
	int action[][][] = new int[11][10][2];


};




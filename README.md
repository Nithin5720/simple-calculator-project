import java.awt.*;
import java.awt.event.ActionEvent;
import java.util.Stack;
import javax.swing.*;

public class ExpressionEvaluatorGUI {
    public static void main(String[] args) {
        JFrame frame = new JFrame("Expression Evaluator");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setSize(400, 300);

        JTextField inputField = new JTextField();
        JTextArea outputArea = new JTextArea(5, 20);
        outputArea.setEditable(false);
        outputArea.setLineWrap(true);
        outputArea.setWrapStyleWord(true);

        JButton evaluateButton = new JButton("Evaluate");
        JComboBox<String> modeSelector = new JComboBox<>(new String[]{"Postfix", "Prefix", "Infix"});

        evaluateButton.addActionListener((ActionEvent e) -> {
            String expression = inputField.getText().trim();
            String mode = (String) modeSelector.getSelectedItem();

            if (expression.isEmpty()) {
                outputArea.setText("Please enter an expression.");
                return;
            }

            try {
                int result = switch (mode) {
                    case "Postfix" -> evaluatePostfix(expression);
                    case "Prefix" -> evaluatePrefix(expression);
                    case "Infix" -> evaluateInfix(expression);
                    default -> throw new IllegalArgumentException("Unsupported mode: " + mode);
                };
                outputArea.setText("Result: " + result);
            } catch (Exception ex) {
                outputArea.setText("Error: " + ex.getMessage());
            }
        });

        frame.setLayout(new BoxLayout(frame.getContentPane(), BoxLayout.Y_AXIS));
        frame.add(new JLabel("Enter Expression:"));
        frame.add(inputField);
        frame.add(new JLabel("Select Mode:"));
        frame.add(modeSelector);
        frame.add(evaluateButton);
        frame.add(new JScrollPane(outputArea));

        frame.setVisible(true);
    }

    private static int evaluatePostfix(String expr) {
        Stack<Integer> stack = new Stack<>();
        String[] tokens = expr.split("\s+");
        for (String token : tokens) {
            if (token.matches("\\d+")) { // Match multi-digit numbers
                stack.push(Integer.parseInt(token));
            } else if ("+-*/^".contains(token)) {
                if (stack.size() < 2) throw new IllegalArgumentException("Invalid Postfix Expression");
                int b = stack.pop();
                int a = stack.pop();
                stack.push(applyOp(token.charAt(0), a, b));
            } else {
                throw new IllegalArgumentException("Invalid token in Postfix Expression: " + token);
            }
        }
        if (stack.size() != 1) throw new IllegalArgumentException("Invalid Postfix Expression");
        return stack.pop();
    }

    private static int evaluatePrefix(String expr) {
        Stack<Integer> stack = new Stack<>();
        String[] tokens = expr.split("\s+");
        for (int i = tokens.length - 1; i >= 0; i--) {
            String token = tokens[i];
            if (token.matches("\\d+")) { // Match multi-digit numbers
                stack.push(Integer.parseInt(token));
            } else if ("+-*/^".contains(token)) {
                if (stack.size() < 2) throw new IllegalArgumentException("Invalid Prefix Expression");
                int a = stack.pop();
                int b = stack.pop();
                stack.push(applyOp(token.charAt(0), a, b));
            } else {
                throw new IllegalArgumentException("Invalid token in Prefix Expression: " + token);
            }
        }
        if (stack.size() != 1) throw new IllegalArgumentException("Invalid Prefix Expression");
        return stack.pop();
    }

    private static int evaluateInfix(String expr) {
        Stack<Integer> values = new Stack<>();
        Stack<Character> ops = new Stack<>();
        for (int i = 0; i < expr.length(); i++) {
            char c = expr.charAt(i);

            if (Character.isDigit(c)) {
                StringBuilder numBuffer = new StringBuilder();
                while (i < expr.length() && Character.isDigit(expr.charAt(i))) {
                    numBuffer.append(expr.charAt(i));
                    i++;
                }
                i--; // Adjust for extra increment
                values.push(Integer.parseInt(numBuffer.toString()));
            } else if (c == '(') {
                ops.push(c);
            } else if (c == ')') {
                while (!ops.isEmpty() && ops.peek() != '(') {
                    values.push(applyOp(ops.pop(), values.pop(), values.pop()));
                }
                if (ops.isEmpty() || ops.pop() != '(') throw new IllegalArgumentException("Mismatched Parentheses");
            } else if ("+-*/^".indexOf(c) != -1) {
                while (!ops.isEmpty() && precedence(ops.peek()) >= precedence(c)) {
                    values.push(applyOp(ops.pop(), values.pop(), values.pop()));
                }
                ops.push(c);
            } else if (!Character.isWhitespace(c)) {
                throw new IllegalArgumentException("Invalid character in Infix Expression: " + c);
            }
        }
        while (!ops.isEmpty()) {
            if (values.size() < 2) throw new IllegalArgumentException("Invalid Infix Expression");
            values.push(applyOp(ops.pop(), values.pop(), values.pop()));
        }
        if (values.size() != 1) throw new IllegalArgumentException("Invalid Infix Expression");
        return values.pop();
    }

    private static int applyOp(char op, int a, int b) {
        return switch (op) {
            case '+' -> a + b;
            case '-' -> a - b;
            case '*' -> a * b;
            case '/' -> {
                if (b == 0) throw new ArithmeticException("Division by Zero");
                yield a / b;
            }
            case '^' -> (int) Math.pow(a, b);
            default -> throw new IllegalArgumentException("Invalid Operator: " + op);
        };
    }

    private static int precedence(char op) {
        return switch (op) {
            case '+', '-' -> 1;
            case '*', '/' -> 2;
            case '^' -> 3;
            default -> 0;
        };
    }
}


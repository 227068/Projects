# Projects

package com.example.calculatorkotlin

import android.annotation.SuppressLint
import android.os.Bundle
import android.view.View
import android.widget.Button
import android.widget.TextView
import androidx.appcompat.app.AppCompatActivity
import kotlin.math.cos
import kotlin.math.pow
import kotlin.math.sin
import kotlin.math.sqrt
import kotlin.math.tan

class MainActivity : AppCompatActivity() {
    private lateinit var textViewResult: TextView

    private var lastNumeric: Boolean = false
    private var lastDot: Boolean = false

    @SuppressLint("MissingInflatedId")
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        textViewResult = findViewById(/* id = */ R.id.textViewResult)

        // Находим все кнопки по ID и устанавливаем слушатели кликов
        val buttons = arrayOf(
            R.id.button0, R.id.button1, R.id.button2, R.id.button3, R.id.button4,
            R.id.button5, R.id.button6, R.id.button7, R.id.button8, R.id.button9,
            R.id.buttonPlus, R.id.buttonMinus, R.id.buttonMultiply, R.id.buttonDivide,
            R.id.buttonDecimal, R.id.buttonClear, R.id.buttonEquals, R.id.buttonPlusMinus,
            R.id.buttonPercent
        )

        for (buttonId in buttons) {
            val button = findViewById<Button>(buttonId)
            button.setOnClickListener { onButtonClick(it) }
        }
    }

    private fun onButtonClick(view: View) {
        when (view.id) {
            R.id.button0, R.id.button1, R.id.button2, R.id.button3, R.id.button4,
            R.id.button5, R.id.button6, R.id.button7, R.id.button8, R.id.button9 -> {
                onDigitClick((view as Button).text.toString())
            }
            R.id.buttonPlus, R.id.buttonMinus, R.id.buttonMultiply, R.id.buttonDivide -> {
                onOperatorClick((view as Button).text.toString())
            }
            R.id.buttonDecimal -> {
                onDecimalClick()
            }
            R.id.buttonClear -> {
                onClearClick()
            }
            R.id.buttonEquals -> {
                onEqualsClick()
            }
            R.id.buttonPlusMinus -> {
                onPlusMinusClick()
            }
            R.id.buttonPercent -> {
                onPercentClick()
            }
        }
    }

    private fun onDigitClick(digit: String) {
        if (textViewResult.text.toString() == "0") {
            textViewResult.text = digit
        } else {
            textViewResult.append(digit)
        }
        lastNumeric = true
        lastDot = false
    }

    private fun onOperatorClick(operator: String) {
        if (lastNumeric && !isOperatorAdded(textViewResult.text.toString())) {
            textViewResult.append(operator)
            lastNumeric = false
            lastDot = false
        }
    }

    private fun onDecimalClick() {
        if (lastNumeric && !lastDot) {
            textViewResult.append(".")
            lastNumeric = false
            lastDot = true
        }
    }

    private fun onClearClick() {
        textViewResult.text = "0"
        lastNumeric = false
        lastDot = false
    }

    @SuppressLint("SetTextI18n")
    private fun onEqualsClick() {
        if (lastNumeric) {
            var value = textViewResult.text.toString()
            try {
                // Replace division and multiplication symbols with symbols that can be interpreted by Eval
                value = value.replace("/", "*")
                value = value.replace("*", "/")

                val result = eval(value)
                textViewResult.text = result.toString()
            } catch (e: ArithmeticException) {
                e.printStackTrace()
                textViewResult.text = "Error"
            }
        }
    }
    private fun onPlusMinusClick() {
        val currentValue = textViewResult.text.toString()
        if (currentValue != "0") {
            if (currentValue.startsWith("-")) {
                textViewResult.text = currentValue.substring(1)
            } else {
                "-$currentValue".also { textViewResult.text = it }
            }
        }
    }

    @SuppressLint("SetTextI18n")
    private fun onPercentClick() {
        if (lastNumeric) {
            try {
                val value = textViewResult.text.toString().toDouble()
                val result = value / 100.0
                result.toString().also { this.textViewResult.text = it }
            } catch (e: NumberFormatException) {
                e.printStackTrace()
                textViewResult.text = "Error"
            }
        }
    }

    private fun isOperatorAdded(value: String): Boolean {
        return value.startsWith("-")  value.contains("+")  value.contains("-")  value.contains("*")  value.contains("/")
    }


    // Function to evaluate the string expression
    private fun eval(str: String): Double {
        return object : Any() {
            var pos = -1
            var ch = 0

            fun nextChar() {
                ch = if ((++pos < str.length)) str[pos].code else -1
            }

            fun eat(charToEat: Int): Boolean {
                while (ch == ' '.code) nextChar()
                if (ch == charToEat) {
                    nextChar()
                    return true
                }
                return false
            }

            fun parse(): Double {
                nextChar()
                val x = parseExpression()
                if (pos < str.length) throw Exception("Unexpected: " + ch.toChar())
                return x
            }

            // Grammar:
            // expression = term | expression + term | expression - term
            // term = factor | term * factor | term / factor
            // factor = + factor | - factor | ( expression )
            //        | number | functionName factor | factor ^ factor

            fun parseExpression(): Double {
                var x = parseTerm()
                while (true) {
                    if (eat('+'.code)) x += parseTerm() // addition
                    else if (eat('-'.code)) x -= parseTerm() // subtraction
                    else return x
                }
            }

            fun parseTerm(): Double {
                var x = parseFactor()
                while (true) {
                    if (eat('*'.code)) x *= parseFactor() // multiplication
                    else if (eat('/'.code)) x /= parseFactor() // division
                    else return x
                }
            }

            fun parseFactor(): Double {
                if (eat('+'.code)) return parseFactor() // unary plus
                if (eat('-'.code)) return -parseFactor() // unary minus

                var x: Double
                val startPos = pos
                if (eat('('.code)) { // parentheses
                    x = parseExpression()
                    eat(')'.code)
                } else if ((ch >= '0'.code && ch <= '9'.code)  ch == '.'.code) { // numbers
                    while ((ch >= '0'.code && ch <= '9'.code)  ch == '.'.code) nextChar()
                    x = str.substring(startPos, pos).toDouble()
                } else if (ch >= 'a'.code && ch <= 'z'.code) { // functions
                    while (ch >= 'a'.code && ch <= 'z'.code) nextChar()
                    val func = str.substring(startPos, pos)
                    x = parseFactor()
                    x =
                        when (func) {
                            "sqrt" -> sqrt(x)
                            "sin" -> sin(Math.toRadians(x))
                            "cos" -> cos(Math.toRadians(x))
                            "tan" -> tan(Math.toRadians(x))
                            else -> throw RuntimeException("Unknown function: $func")
                        }
                } else {
                    throw Exception("Unexpected: " + ch.toChar())
                }
                if (eat('^'.code)) x = x.pow(parseFactor()) // exponentiation

                return x
            }
        }.parse()
    }
}

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/textViewResult"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="36sp"
        android:gravity="end"
        android:padding="8dp"
        android:background="#EEEEEE"
        android:textColor="#000000"
        android:text="0"/>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal">

            <Button
                android:id="@+id/buttonClear"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="C"
                android:textSize="24sp"
                android:layout_margin="4dp"/>

            <Button
                android:id="@+id/buttonPlusMinus"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="+/-"
                android:textSize="24sp"
                android:layout_margin="4dp"/>

            <Button
                android:id="@+id/buttonPercent"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="%"
                android:textSize="24sp"
                android:layout_margin="4dp"/>

            <Button
                android:id="@+id/buttonDivide"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="/"
                android:textSize="24sp"
                android:layout_margin="4dp"/>
        </LinearLayout>

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal">

            <Button
                android:id="@+id/button7"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="7"
                android:textSize="24sp"
                android:layout_margin="4dp"/>

            <Button
                android:id="@+id/button8"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="8"
                android:textSize="24sp"
                android:layout_margin="4dp"/>

            <Button
                android:id="@+id/button9"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="9"
                android:textSize="24sp"
                android:layout_margin="4dp"/>

            <Button
                android:id="@+id/buttonMultiply"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="*"
                android:textSize="24sp"
                android:layout_margin="4dp"/>
        </LinearLayout>

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal">
            <Button
                android:id="@+id/button4"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="4"
                android:textSize="24sp"
                android:layout_margin="4dp"/>

            <Button
                android:id="@+id/button5"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="5"
                android:textSize="24sp"
                android:layout_margin="4dp"/>

            <Button
                android:id="@+id/button6"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="6"
                android:textSize="24sp"
                android:layout_margin="4dp"/>

            <Button
                android:id="@+id/buttonMinus"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="-"
                android:textSize="24sp"
                android:layout_margin="4dp"/>
        </LinearLayout>

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal">

            <Button
                android:id="@+id/button1"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="1"
                android:textSize="24sp"
                android:layout_margin="4dp"/>

            <Button
                android:id="@+id/button2"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="2"
                android:textSize="24sp"
                android:layout_margin="4dp"/>

            <Button
                android:id="@+id/button3"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="3"
                android:textSize="24sp"
                android:layout_margin="4dp"/>

            <Button
                android:id="@+id/buttonPlus"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="+"
                android:textSize="24sp"
                android:layout_margin="4dp"/>
        </LinearLayout>

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal">

            <Button
                android:id="@+id/button0"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="2"
                android:text="0"
                android:textSize="24sp"
                android:layout_margin="4dp"/>

            <Button
                android:id="@+id/buttonDecimal"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="."
                android:textSize="24sp"
                android:layout_margin="4dp"/>

            <Button
                android:id="@+id/buttonEquals"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="="
                android:textSize="24sp"
                android:layout_margin="4dp"/>
        </LinearLayout>

    </LinearLayout>

</LinearLayout>

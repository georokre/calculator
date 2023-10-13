<script lang="ts">
    import Button from '@smui/button';
    import { create, all } from 'mathjs';
    const config = { }
    const math = create(all, config)

    //the expression to be evaluated
    let expression = '0'

    //the current lower display
    let output = '0'

    //the current upper display
    let ansString = ''

    //the previous answer value
    let ansValue = ''

    //error message if input entered is illegal
    let error = ''

    //number of current open parentheseses
    let openParentheses = 0

    //add a number to the expression
    function addNumber(x:string) {

        //if expression is in initial state, replace it with the entered number
        if (expression == '0'){
            //if answer is not on top yet, put it there
            if (ansValue != ''){
                ansString = 'ans = ' + ansValue
            }
            expression = x
        }
        //otherwise add it to the end of the expression
        else {
            expression += x
        }
        output = expression
    }

    //add a basic operation to the display
    function addBasicOpp(x:string) {
        //if answer is not on top yet, put it there
        if (ansValue != '' && expression == '0'){
            ansString = 'ans = ' + ansValue
            expression = 'ans'
        }
        //check if it is valid to add a basic operation
        let prevVal = expression.slice(-1)
        if (expression != '0' && prevVal != '(' && prevVal != ' '){
            expression += ' ' + x + ' '
        }
        
        output = expression
    }

    //add a decimal point
    function addDecimal() {
        
        let prevVal = expression.split(" ").slice(-1)[0]

        //check if previous input is a number that doesn't contain a decimal already
        if (!prevVal.includes('.')){
            if (prevVal == ''){
                if (ansValue != ''){
                    ansString = 'ans = ' + ansValue
                }
                
                expression += '0.'
            }
            else {
                expression += '.'
            }
            
        }
        output = expression
    }

    //add a special function to the expression
    function addFunction(x:string) {

        //if answer is not on top yet move it there
        if (ansValue != '' && expression == '0'){
            ansString = 'ans = ' + ansValue
        }
        //if expression is in initial state, set expression to the function and open a parenthesis
        if (expression == '0'){
            expression = x + '('
        }

        //otherwise add the function to the end of the expression and open a parenthesis
        else {
            expression += x + '('
        }
        openParentheses += 1
        
        output = expression
    }

    //insert the previous answer into the expression
    function addAns() {
        let prevVal = expression.slice(-1)
        //if the last entered input is a function or open parenthesis and there is an answer value add it to the expression
        if ((prevVal == ' ' || prevVal == '(') && ansValue != ''){
            expression += 'ans'
            output = expression
        }
        
    }

    //add an opening parenthesis
    function openParenthesis() {

        //if state isn't in initial state add a parenthesis to the end
        if (expression != '0'){
            expression += '('
        }
        //if it is in the intial state, replace the expression with the opening parenthesis
        else {
            expression = '('
        }
        output = expression
        openParentheses += 1

    }

    //add a closing parenthesis
    function closeParenthesis() {
        let prevVal = expression.split(" ").slice(-1)[0]

        //check if last thing entered is an operation or if there are any open parentheses, only allow a closing if these are true
        if (prevVal != '' && openParentheses != 0){
            expression += ')'
            openParentheses -= 1
        }
        output = expression
    }

    //deletes last character entered
    function backspace(){

        //trims added white space at end (if any) then slices the last character then trims added white space at end (if any) again
        expression = expression.trimEnd().slice(0,-1).trimEnd();
        
        //if expression is now empty set display to 0
        if (expression == ''){
            //if answer is not on top yet move it there
            if (ansString != ''){
                ansString = 'ans = ' + ansValue
            }
            expression = '0'
        }
        output = expression
    }



    //calculates the expression entered
    function calculate() {

        //closes all open parentheses
        while (openParentheses != 0){
            expression += ')'
            openParentheses -= 1
        }
        output = expression

        //resets error message
        error = ''

        //attempts to evaluate the expression entered
        try {
            // if it succedes in calculating display the answer in the bottom and the equation entered on the top of the display
            ansString = output + " = "
            ansString = ansString.replaceAll('ans',ansValue)
            ansValue = math.evaluate(expression,{ans:ansValue})
            output = ansValue
            expression = '0'
        }
        catch(e) {
            if (e instanceof Error){
                error = e.message
            }
        }
    }




    //All clear button on calculator, resets everything to intitial state
    function clear() {
        expression = '0'
        output = '0'
        ansString = ''
        ansValue = ''
        error = ''
    }

    

    
    
    
    // call functions from keyboard input
    function onKeyDown(e:KeyboardEvent) {
        const nums = ['0','1','2','3','4','5','6','7','8','9']
        const basicOpps = ['+','-','*','/','^']
        if (nums.includes(e.key)){
        addNumber(e.key)
        }
        else if (basicOpps.includes(e.key)){
        addBasicOpp(e.key)
        }
        else if (e.key == "Enter"){
        calculate()
        }
        else if (e.key == "Backspace"){
        backspace()
        }
        else if (e.key == "."){
        addDecimal()
        }
        else if (e.key == "("){
        openParenthesis()
        }
        else if (e.key == ")"){
        closeParenthesis()
        }
	}
</script>

<div id="calculator">
    <div class='calcRow'>
        <div id='display'>
            <div id="ans">
                {ansString}
            </div>
            <div id="output">{output}</div>
        </div>
    </div>

    <div class='calcRow'>
        <Button class="calcButton" on:click={() => openParenthesis()} variant="raised">(</Button>
        <Button class="calcButton" on:click={() => closeParenthesis()} variant="raised">)</Button>
        <Button class="calcButton" on:click={() => backspace()} variant="raised">CE</Button>
        <Button class="calcButton" on:click={() => clear()} variant="raised">AC</Button>
    </div>
    <div class='calcRow'>
        <Button class="calcButton" on:click={() => addAns()} variant="raised">ans</Button>
        <Button class="calcButton" on:click={() => addFunction('log')} variant="raised">log</Button>
        <Button class="calcButton" on:click={() => addFunction('sqrt')} variant="raised">sqrt</Button>
        <Button class="calcButton" on:click={() => addBasicOpp('^')} variant="raised">^</Button>
    </div>
    <div class='calcRow'>
        <Button class="calcButton" on:click={() => addNumber('7')} variant="raised">7</Button>
        <Button class="calcButton" on:click={() => addNumber('8')} variant="raised">8</Button>
        <Button class="calcButton" on:click={() => addNumber('9')} variant="raised">9</Button>
        <Button class="calcButton" on:click={() => addBasicOpp('/')} variant="raised">/</Button>
    </div>
    <div class='calcRow'>
        <Button class="calcButton" on:click={() => addNumber('4')} variant="raised">4</Button>
        <Button class="calcButton" on:click={() => addNumber('5')} variant="raised">5</Button>
        <Button class="calcButton" on:click={() => addNumber('6')} variant="raised">6</Button>
        <Button class="calcButton" on:click={() => addBasicOpp('*')} variant="raised">*</Button>
    </div>
    <div class='calcRow'>
        <Button class="calcButton" on:click={() => addNumber('1')} variant="raised">1</Button>
        <Button class="calcButton" on:click={() => addNumber('2')} variant="raised">2</Button>
        <Button class="calcButton" on:click={() => addNumber('3')} variant="raised">3</Button>
        <Button class="calcButton" on:click={() => addBasicOpp('-')} variant="raised">-</Button>
    </div>
    <div class='calcRow'>
        <Button class="calcButton" on:click={() => addNumber('0')} variant="raised">0</Button>
        <Button class="calcButton" on:click={() => addDecimal()} variant="raised">.</Button>
        <Button class="calcButton" on:click={() => calculate()} variant="raised">=</Button>
        <Button class="calcButton" on:click={() => addBasicOpp('+')} variant="raised">+</Button>
    </div>
    {#if error !=''}
    <div>Error: {error}</div>
    {/if}
</div>



<svelte:window on:keydown|preventDefault={onKeyDown} />


<style>
    * :global(.calcButton) {
        height: 50px;
        width: 75px;
        margin: 5px;
    }
    
    
    #ans {
        
        height:25px;
        line-height:25px;
        float: right;
    } 
    #output {
        
        height:50px;
        line-height:50px;
        font-size: 24px;
        float:right;
        clear:right;
    }
    
    .calcRow{
        display: flex;
        width: 500px;
        
    }
</style>
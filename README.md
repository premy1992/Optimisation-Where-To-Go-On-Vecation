{
 "metadata": {
  "name": "",
  "signature": "sha256:dcf21d85beca808bacb64ffc0026f3a88a6ecd4c1d6260e5f3f6fc200cf0df9a"
 },
 "nbformat": 3,
 "nbformat_minor": 0,
 "worksheets": [
  {
   "cells": [
    {
     "cell_type": "markdown",
     "metadata": {},
     "source": [
      "#Linear Programing In Python : Deciding Where to Go on Vacation\n",
      "\n",
      "This iPython Notebook is an Example of Constructing a Linear Program in Python with PULP module.\n",
      "\n",
      "----------\n",
      "**Problem Formulation:**\n",
      "You want to go on vacation, but you have only limited number of days. On top of it, we also want keep the cost at minimum.\n",
      "And the internet offers plenty of options how to decide, so which packages/mix of packages should we select? \n",
      "\n",
      "----------\n",
      "> - Objective: Minimize Cost of Vacation while selecting Optimal Vacation Package\n",
      "> - LP Form: Minimization\n",
      "> - Decision Variables: Binary Variables whether to purchase the package or not.\n",
      "> - Constrains: Limited Number of Vacation"
     ]
    },
    {
     "cell_type": "code",
     "collapsed": false,
     "input": [
      "from pulp import *\n",
      "import numpy as np\n",
      "import pandas as pd\n",
      "import re \n",
      "import matplotlib.pyplot as plt\n",
      "from IPython.display import Image\n",
      "%matplotlib inline"
     ],
     "language": "python",
     "metadata": {},
     "outputs": [],
     "prompt_number": 1
    },
    {
     "cell_type": "markdown",
     "metadata": {},
     "source": [
      "**Getting the Data**\n",
      ">There are multiple websites that provide full-priced and discount vacation packages.\n",
      "The dataset from this problem was scraped from The Clymb Adventures http://www.theclymb.com/adventures \n",
      "\n",
      "![TheClymb](https://photos-1.dropbox.com/t/2/AAB8eX8O_-HLLEXt482rsjiDDj-Cy-mvF1DZT6MjP5GKVg/12/49846494/png/32x32/1/_/1/2/thclymb.png/ENCbqCYY-KgMIAEoAQ/S5J_1b9un4Uy-zuWdfU7bKaGgVECPSAFNnMgrfDttqA?size=1024x768&size_mode=2)\n"
     ]
    },
    {
     "cell_type": "markdown",
     "metadata": {},
     "source": [
      "**Understanding the Dataset**\n",
      ">The dataset contains:\n",
      "> - Final Destination\n",
      "> - Duration of the trip\n",
      "> - Total Cost of the trip\n",
      "> - Short Description of the adventure"
     ]
    },
    {
     "cell_type": "code",
     "collapsed": false,
     "input": [
      "data = pd.read_csv('clymb_adventures.csv')\n",
      "data[:5]"
     ],
     "language": "python",
     "metadata": {},
     "outputs": [
      {
       "html": [
        "<div style=\"max-height:1000px;max-width:1500px;overflow:auto;\">\n",
        "<table border=\"1\" class=\"dataframe\">\n",
        "  <thead>\n",
        "    <tr style=\"text-align: right;\">\n",
        "      <th></th>\n",
        "      <th>destination</th>\n",
        "      <th>duration</th>\n",
        "      <th>cost</th>\n",
        "      <th>description</th>\n",
        "    </tr>\n",
        "  </thead>\n",
        "  <tbody>\n",
        "    <tr>\n",
        "      <th>0</th>\n",
        "      <td>     Baja</td>\n",
        "      <td>  7</td>\n",
        "      <td>  899</td>\n",
        "      <td> Hike Bike and Sea Kayak and more on a Remote P...</td>\n",
        "    </tr>\n",
        "    <tr>\n",
        "      <th>1</th>\n",
        "      <td>    Nepal</td>\n",
        "      <td> 11</td>\n",
        "      <td>  899</td>\n",
        "      <td> Explore the land and culture of the Himalayas....</td>\n",
        "    </tr>\n",
        "    <tr>\n",
        "      <th>2</th>\n",
        "      <td>    Spain</td>\n",
        "      <td>  8</td>\n",
        "      <td>  568</td>\n",
        "      <td> Sport climb &amp; deep water solo in Mallorca. Dis...</td>\n",
        "    </tr>\n",
        "    <tr>\n",
        "      <th>3</th>\n",
        "      <td> Yosemite</td>\n",
        "      <td>  5</td>\n",
        "      <td>  750</td>\n",
        "      <td> Guided hiking through stunning high country. E...</td>\n",
        "    </tr>\n",
        "    <tr>\n",
        "      <th>4</th>\n",
        "      <td>     Utah</td>\n",
        "      <td>  6</td>\n",
        "      <td> 1356</td>\n",
        "      <td> Hike. Canyoneer. Rock Climb. and more!. Experi...</td>\n",
        "    </tr>\n",
        "  </tbody>\n",
        "</table>\n",
        "</div>"
       ],
       "metadata": {},
       "output_type": "pyout",
       "prompt_number": 2,
       "text": [
        "  destination  duration  cost  \\\n",
        "0        Baja         7   899   \n",
        "1       Nepal        11   899   \n",
        "2       Spain         8   568   \n",
        "3    Yosemite         5   750   \n",
        "4        Utah         6  1356   \n",
        "\n",
        "                                         description  \n",
        "0  Hike Bike and Sea Kayak and more on a Remote P...  \n",
        "1  Explore the land and culture of the Himalayas....  \n",
        "2  Sport climb & deep water solo in Mallorca. Dis...  \n",
        "3  Guided hiking through stunning high country. E...  \n",
        "4  Hike. Canyoneer. Rock Climb. and more!. Experi...  "
       ]
      }
     ],
     "prompt_number": 2
    },
    {
     "cell_type": "code",
     "collapsed": false,
     "input": [
      "fig, axs = plt.subplots(1,2)\n",
      "my_plot = data[['destination', 'cost']].plot(kind='bar', title=\"Destination by Cost\", ax=axs[0])\n",
      "my_plot = data[['destination', 'duration']].plot(kind='bar', title=\"Destination by Duration\", ax=axs[1])"
     ],
     "language": "python",
     "metadata": {},
     "outputs": [
      {
       "metadata": {},
       "output_type": "display_data",
       "png": "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\nvn1qGKmzw/Tp09P2oZvLL798+vTp02lpaaG1tbU7P5iOKotTN6l20rDZyL6naXP27NlMnz79clKg\nWGxntZ+aLZYvv/xy3BdiZ1NufGnU47z88ssriu3Ib9rWGxHRLPljNBcigib70jaObYvtlHCvDRWQ\nWHp3I1FpbGfyt3RyuVzJdFRZnLpJtZOGzWJlImKfwKdcX6dF+Dym3U/94eNwfyuJjSxf++XqlqPi\nZZlG9mnW2Utcei7yxsDOW/1otNhIGpN0mgSR5n1cjUupvvD5mZJ07LzVl2bt70pjO5OSjmEYhpE8\nmRzwm0lTS6MdI7t91Bf91UiGRr72y9UtRyYHfMMwssfUqVO55JJL6mLrl7/8JSeeeGJdbPUnTMNv\nEoppk/V4QZXG+Wpra2Py5Ml86UtfKlre6Bp+Vs/bmWeeyejRo7niiisS9aWzs5P3ve99dHV1MWBA\nbeeg/V3Dt1U6TU8tgzudFQ/9Y6VFNs9bNYPl9u3bKxrIm3EgzhqZlHSaSVMzDd+xdOlSPv3pT7PX\nXnsxbNgwzj33XFSV7373u7S2tjJ8+HCmTJnC+vXrAdi8eTNf+MIXGDZsGEOHDmXcuHGsXr2ab3/7\n2zz00EOcc845DBkyhPPOC/97U0dW+6iRNPwFCxZw2GGHsdtuuzFp0iQ2b94MwKxZsxg/fnxB3QED\nBvDyyy8DTvr52te+xic+8QkGDx5MLpfj7rvv5tBDD2X33XdnzJgx/tuvjo99zP1nwpaWFnbbbTce\nf/zxXjYeffRRjjjiCFpaWhg3bhyPPfZYd1lbWxuXXnopRx99NLvtthsnnngia9asKXlcjXztl6tb\njkwO+EZzsW3bNk499VT2228/Xn31VVasWMGkSZO46aabmD17NrlcjpdffpmNGzdyzjnnADB79mzW\nr1/PsmXLWLt2LT//+c/ZZZdduPLKKxk/fjw//elP2bBhA9dee23KR9ecbNmyhYkTJzJlyhTWrVvH\nZz/7WX71q1+FvsxUmrlz53LJJZewceNGPvrRjzJ48GBuvvlm3nrrLe6++26uu+467rzzTgAeeugh\nAN566y3Wr1/Phz/84YK21q5dyymnnMLXv/511q5dywUXXMApp5zCunXrCuzNmjWL1atXs2XLFn74\nwx8m2BtNRPBnN9P+OHeMaijWd4CC1vBT2fl69NFHdc8999Rt27YV5B977LF63XXXdaeff/553XHH\n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\n",
       "text": [
        "<matplotlib.figure.Figure at 0x10d99d650>"
       ]
      }
     ],
     "prompt_number": 3
    },
    {
     "cell_type": "markdown",
     "metadata": {},
     "source": [
      "**Setting Up LP Problem:**\n",
      "\n",
      "> Define The LP Object\n",
      "\n",
      "> The *prob* variable is created to contain the formulation, and the usual parameters are passed into LpProblem."
     ]
    },
    {
     "cell_type": "code",
     "collapsed": false,
     "input": [
      "# create the LP object, set up as a minimization problem --> since we want to minimize the costs \n",
      "prob = pulp.LpProblem('GoingOnVacation', pulp.LpMinimize)"
     ],
     "language": "python",
     "metadata": {},
     "outputs": [],
     "prompt_number": 4
    },
    {
     "cell_type": "markdown",
     "metadata": {},
     "source": [
      "> Create Decision Variables:"
     ]
    },
    {
     "cell_type": "code",
     "collapsed": false,
     "input": [
      "decision_variables = []\n",
      "for rownum, row in data.iterrows():\n",
      "\tvariable = str('x' + str(rownum))\n",
      "\tvariable = pulp.LpVariable(str(variable), lowBound = 0, upBound = 1, cat= 'Integer') #make variables binary\n",
      "\tdecision_variables.append(variable)\n",
      "\n",
      "print (\"Total number of decision_variables: \" + str(len(decision_variables)))\n",
      "print (\"Array with Decision Variables:\" + str(decision_variables))"
     ],
     "language": "python",
     "metadata": {},
     "outputs": [
      {
       "output_type": "stream",
       "stream": "stdout",
       "text": [
        "Total number of decision_variables: 42\n",
        "Array with Decision Variables:[x0, x1, x2, x3, x4, x5, x6, x7, x8, x9, x10, x11, x12, x13, x14, x15, x16, x17, x18, x19, x20, x21, x22, x23, x24, x25, x26, x27, x28, x29, x30, x31, x32, x33, x34, x35, x36, x37, x38, x39, x40, x41]\n"
       ]
      }
     ],
     "prompt_number": 5
    },
    {
     "cell_type": "markdown",
     "metadata": {},
     "source": [
      "> Define Objective Function: (*Minimixing the Cost of The Trip*)\n",
      "\n",
      "> The variable prob now begins collecting problem data with the += operator. The objective function is logically entered first, with an important comma , at the end of the statement and a short string explaining what this objective function is:"
     ]
    },
    {
     "cell_type": "code",
     "collapsed": false,
     "input": [
      "total_cost = \"\"\n",
      "for rownum, row in data.iterrows():\n",
      "\tfor i, schedule in enumerate(decision_variables):\n",
      "\t\tif rownum == i:\n",
      "\t\t\tformula = row['cost']*schedule\n",
      "\t\t\ttotal_cost += formula\n",
      "\n",
      "prob += total_cost\n",
      "print (\"Optimization function: \" + str(total_cost))\t"
     ],
     "language": "python",
     "metadata": {},
     "outputs": [
      {
       "output_type": "stream",
       "stream": "stdout",
       "text": [
        "Optimization function: 899*x0 + 899*x1 + 399*x10 + 385*x11 + 439*x12 + 339*x13 + 399*x14 + 265*x15 + 849*x16 + 1799*x17 + 1799*x18 + 1499*x19 + 568*x2 + 3175*x20 + 599*x21 + 1450*x22 + 399*x23 + 1199*x24 + 2799*x25 + 2675*x26 + 1699*x27 + 599*x28 + 1798*x29 + 750*x3 + 1799*x30 + 1199*x31 + 999*x32 + 1375*x33 + 1199*x34 + 299*x35 + 2898*x36 + 499*x37 + 1499*x38 + 450*x39 + 1356*x4 + 198*x40 + 375*x41 + 680*x5 + 559*x6 + 899*x7 + 1799*x8 + 1625*x9\n"
       ]
      }
     ],
     "prompt_number": 6
    },
    {
     "cell_type": "markdown",
     "metadata": {},
     "source": [
      "> Define Constrains: (*Selected Packages should not exceed total vacation days available*)"
     ]
    },
    {
     "cell_type": "code",
     "collapsed": false,
     "input": [
      "aval_vacation_days = 10\n",
      "total_vacation_days = \"\"\n",
      "for rownum, row in data.iterrows():\n",
      "\tfor i, schedule in enumerate(decision_variables):\n",
      "\t\tif rownum == i:\n",
      "\t\t\tformula = row['duration']*schedule\n",
      "\t\t\ttotal_vacation_days += formula\n",
      "\n",
      "prob += (total_vacation_days == aval_vacation_days)"
     ],
     "language": "python",
     "metadata": {},
     "outputs": [],
     "prompt_number": 7
    },
    {
     "cell_type": "markdown",
     "metadata": {},
     "source": [
      ">The Final Format"
     ]
    },
    {
     "cell_type": "code",
     "collapsed": false,
     "input": [
      "print prob\n",
      "prob.writeLP(\"GoingOnVacation.lp\" )"
     ],
     "language": "python",
     "metadata": {},
     "outputs": [
      {
       "output_type": "stream",
       "stream": "stdout",
       "text": [
        "GoingOnVacation:\n",
        "MINIMIZE\n",
        "899*x0 + 899*x1 + 399*x10 + 385*x11 + 439*x12 + 339*x13 + 399*x14 + 265*x15 + 849*x16 + 1799*x17 + 1799*x18 + 1499*x19 + 568*x2 + 3175*x20 + 599*x21 + 1450*x22 + 399*x23 + 1199*x24 + 2799*x25 + 2675*x26 + 1699*x27 + 599*x28 + 1798*x29 + 750*x3 + 1799*x30 + 1199*x31 + 999*x32 + 1375*x33 + 1199*x34 + 299*x35 + 2898*x36 + 499*x37 + 1499*x38 + 450*x39 + 1356*x4 + 198*x40 + 375*x41 + 680*x5 + 559*x6 + 899*x7 + 1799*x8 + 1625*x9 + 0\n",
        "SUBJECT TO\n",
        "_C1: 7 x0 + 11 x1 + 2 x10 + 2 x11 + 3 x12 + 2 x13 + 3 x14 + 4 x15 + 7 x16\n",
        " + 8 x17 + 10 x18 + 7 x19 + 8 x2 + 13 x20 + 3 x21 + 5 x22 + 2 x23 + 5 x24\n",
        " + 9 x25 + 14 x26 + 8 x27 + 4 x28 + 6 x29 + 5 x3 + 8 x30 + 11 x31 + 8 x32\n",
        " + 8 x33 + 13 x34 + 4 x35 + 6 x36 + 3 x37 + 5 x38 + 4 x39 + 6 x4 + 2 x40\n",
        " + 2 x41 + 4 x5 + 4 x6 + 6 x7 + 10 x8 + 7 x9 = 10\n",
        "\n",
        "VARIABLES\n",
        "0 <= x0 <= 1 Integer\n",
        "0 <= x1 <= 1 Integer\n",
        "0 <= x10 <= 1 Integer\n",
        "0 <= x11 <= 1 Integer\n",
        "0 <= x12 <= 1 Integer\n",
        "0 <= x13 <= 1 Integer\n",
        "0 <= x14 <= 1 Integer\n",
        "0 <= x15 <= 1 Integer\n",
        "0 <= x16 <= 1 Integer\n",
        "0 <= x17 <= 1 Integer\n",
        "0 <= x18 <= 1 Integer\n",
        "0 <= x19 <= 1 Integer\n",
        "0 <= x2 <= 1 Integer\n",
        "0 <= x20 <= 1 Integer\n",
        "0 <= x21 <= 1 Integer\n",
        "0 <= x22 <= 1 Integer\n",
        "0 <= x23 <= 1 Integer\n",
        "0 <= x24 <= 1 Integer\n",
        "0 <= x25 <= 1 Integer\n",
        "0 <= x26 <= 1 Integer\n",
        "0 <= x27 <= 1 Integer\n",
        "0 <= x28 <= 1 Integer\n",
        "0 <= x29 <= 1 Integer\n",
        "0 <= x3 <= 1 Integer\n",
        "0 <= x30 <= 1 Integer\n",
        "0 <= x31 <= 1 Integer\n",
        "0 <= x32 <= 1 Integer\n",
        "0 <= x33 <= 1 Integer\n",
        "0 <= x34 <= 1 Integer\n",
        "0 <= x35 <= 1 Integer\n",
        "0 <= x36 <= 1 Integer\n",
        "0 <= x37 <= 1 Integer\n",
        "0 <= x38 <= 1 Integer\n",
        "0 <= x39 <= 1 Integer\n",
        "0 <= x4 <= 1 Integer\n",
        "0 <= x40 <= 1 Integer\n",
        "0 <= x41 <= 1 Integer\n",
        "0 <= x5 <= 1 Integer\n",
        "0 <= x6 <= 1 Integer\n",
        "0 <= x7 <= 1 Integer\n",
        "0 <= x8 <= 1 Integer\n",
        "0 <= x9 <= 1 Integer\n",
        "\n"
       ]
      }
     ],
     "prompt_number": 8
    },
    {
     "cell_type": "markdown",
     "metadata": {},
     "source": [
      "> The Actual Optimization:"
     ]
    },
    {
     "cell_type": "code",
     "collapsed": false,
     "input": [
      "\n",
      "optimization_result = prob.solve()\n",
      "\n",
      "assert optimization_result == pulp.LpStatusOptimal\n",
      "print(\"Status:\", LpStatus[prob.status])\n",
      "print(\"Optimal Solution to the problem: \", value(prob.objective))\n",
      "print (\"Individual decision_variables: \")\n",
      "for v in prob.variables():\n",
      "\tprint(v.name, \"=\", v.varValue)"
     ],
     "language": "python",
     "metadata": {},
     "outputs": [
      {
       "output_type": "stream",
       "stream": "stdout",
       "text": [
        "('Status:', 'Optimal')\n",
        "('Optimal Solution to the problem: ', 762.0)\n",
        "Individual decision_variables: \n",
        "('x0', '=', 0.0)\n",
        "('x1', '=', 0.0)\n",
        "('x10', '=', 0.0)\n",
        "('x11', '=', 0.0)\n",
        "('x12', '=', 0.0)\n",
        "('x13', '=', 0.0)\n",
        "('x14', '=', 0.0)\n",
        "('x15', '=', 1.0)\n",
        "('x16', '=', 0.0)\n",
        "('x17', '=', 0.0)\n",
        "('x18', '=', 0.0)\n",
        "('x19', '=', 0.0)\n",
        "('x2', '=', 0.0)\n",
        "('x20', '=', 0.0)\n",
        "('x21', '=', 0.0)\n",
        "('x22', '=', 0.0)\n",
        "('x23', '=', 0.0)\n",
        "('x24', '=', 0.0)\n",
        "('x25', '=', 0.0)\n",
        "('x26', '=', 0.0)\n",
        "('x27', '=', 0.0)\n",
        "('x28', '=', 0.0)\n",
        "('x29', '=', 0.0)\n",
        "('x3', '=', 0.0)\n",
        "('x30', '=', 0.0)\n",
        "('x31', '=', 0.0)\n",
        "('x32', '=', 0.0)\n",
        "('x33', '=', 0.0)\n",
        "('x34', '=', 0.0)\n",
        "('x35', '=', 1.0)\n",
        "('x36', '=', 0.0)\n",
        "('x37', '=', 0.0)\n",
        "('x38', '=', 0.0)\n",
        "('x39', '=', 0.0)\n",
        "('x4', '=', 0.0)\n",
        "('x40', '=', 1.0)\n",
        "('x41', '=', 0.0)\n",
        "('x5', '=', 0.0)\n",
        "('x6', '=', 0.0)\n",
        "('x7', '=', 0.0)\n",
        "('x8', '=', 0.0)\n",
        "('x9', '=', 0.0)\n"
       ]
      }
     ],
     "prompt_number": 9
    },
    {
     "cell_type": "markdown",
     "metadata": {},
     "source": [
      "> The results are stored \n",
      "> If you don't pass the names to the variables and want to append the decision variables back in your desired file format, you want to loop through variable name object. \n",
      "\n",
      "> Depending on your initial data format you might want to parse the results differently. Since in this example we have used pandas dataframe, we will use the number of the variable as index to append the results back to initial dataset"
     ]
    },
    {
     "cell_type": "code",
     "collapsed": false,
     "input": [
      "variable_name = []\n",
      "variable_value = []\n",
      "\n",
      "for v in prob.variables():\n",
      "\tvariable_name.append(v.name)\n",
      "\tvariable_value.append(v.varValue)\n",
      "\n",
      "df = pd.DataFrame({'variable': variable_name, 'value': variable_value})\n",
      "for rownum, row in df.iterrows():\n",
      "\tvalue = re.findall(r'(\\d+)', row['variable'])\n",
      "\tdf.loc[rownum, 'variable'] = int(value[0])\n",
      "\n",
      "df = df.sort_index(by='variable')\n",
      "\n",
      "#append results\n",
      "for rownum, row in data.iterrows():\n",
      "\tfor results_rownum, results_row in df.iterrows():\n",
      "\t\tif rownum == results_row['variable']:\n",
      "\t\t\tdata.loc[rownum, 'decision'] = results_row['value']\n",
      "            \n",
      "data[:5]"
     ],
     "language": "python",
     "metadata": {},
     "outputs": [
      {
       "html": [
        "<div style=\"max-height:1000px;max-width:1500px;overflow:auto;\">\n",
        "<table border=\"1\" class=\"dataframe\">\n",
        "  <thead>\n",
        "    <tr style=\"text-align: right;\">\n",
        "      <th></th>\n",
        "      <th>destination</th>\n",
        "      <th>duration</th>\n",
        "      <th>cost</th>\n",
        "      <th>description</th>\n",
        "      <th>decision</th>\n",
        "    </tr>\n",
        "  </thead>\n",
        "  <tbody>\n",
        "    <tr>\n",
        "      <th>0</th>\n",
        "      <td>     Baja</td>\n",
        "      <td>  7</td>\n",
        "      <td>  899</td>\n",
        "      <td> Hike Bike and Sea Kayak and more on a Remote P...</td>\n",
        "      <td> 0</td>\n",
        "    </tr>\n",
        "    <tr>\n",
        "      <th>1</th>\n",
        "      <td>    Nepal</td>\n",
        "      <td> 11</td>\n",
        "      <td>  899</td>\n",
        "      <td> Explore the land and culture of the Himalayas....</td>\n",
        "      <td> 0</td>\n",
        "    </tr>\n",
        "    <tr>\n",
        "      <th>2</th>\n",
        "      <td>    Spain</td>\n",
        "      <td>  8</td>\n",
        "      <td>  568</td>\n",
        "      <td> Sport climb &amp; deep water solo in Mallorca. Dis...</td>\n",
        "      <td> 0</td>\n",
        "    </tr>\n",
        "    <tr>\n",
        "      <th>3</th>\n",
        "      <td> Yosemite</td>\n",
        "      <td>  5</td>\n",
        "      <td>  750</td>\n",
        "      <td> Guided hiking through stunning high country. E...</td>\n",
        "      <td> 0</td>\n",
        "    </tr>\n",
        "    <tr>\n",
        "      <th>4</th>\n",
        "      <td>     Utah</td>\n",
        "      <td>  6</td>\n",
        "      <td> 1356</td>\n",
        "      <td> Hike. Canyoneer. Rock Climb. and more!. Experi...</td>\n",
        "      <td> 0</td>\n",
        "    </tr>\n",
        "  </tbody>\n",
        "</table>\n",
        "</div>"
       ],
       "metadata": {},
       "output_type": "pyout",
       "prompt_number": 10,
       "text": [
        "  destination  duration  cost  \\\n",
        "0        Baja         7   899   \n",
        "1       Nepal        11   899   \n",
        "2       Spain         8   568   \n",
        "3    Yosemite         5   750   \n",
        "4        Utah         6  1356   \n",
        "\n",
        "                                         description  decision  \n",
        "0  Hike Bike and Sea Kayak and more on a Remote P...         0  \n",
        "1  Explore the land and culture of the Himalayas....         0  \n",
        "2  Sport climb & deep water solo in Mallorca. Dis...         0  \n",
        "3  Guided hiking through stunning high country. E...         0  \n",
        "4  Hike. Canyoneer. Rock Climb. and more!. Experi...         0  "
       ]
      }
     ],
     "prompt_number": 10
    },
    {
     "cell_type": "markdown",
     "metadata": {},
     "source": [
      "> The Final Decisions and Results of the Optimization in the \"User Friendly Way\":"
     ]
    },
    {
     "cell_type": "code",
     "collapsed": false,
     "input": [
      "data[data['decision'] == 1]"
     ],
     "language": "python",
     "metadata": {},
     "outputs": [
      {
       "html": [
        "<div style=\"max-height:1000px;max-width:1500px;overflow:auto;\">\n",
        "<table border=\"1\" class=\"dataframe\">\n",
        "  <thead>\n",
        "    <tr style=\"text-align: right;\">\n",
        "      <th></th>\n",
        "      <th>destination</th>\n",
        "      <th>duration</th>\n",
        "      <th>cost</th>\n",
        "      <th>description</th>\n",
        "      <th>decision</th>\n",
        "    </tr>\n",
        "  </thead>\n",
        "  <tbody>\n",
        "    <tr>\n",
        "      <th>15</th>\n",
        "      <td>  Maine</td>\n",
        "      <td> 4</td>\n",
        "      <td> 265</td>\n",
        "      <td> Ride endless singletrack. Escape to Western Ma...</td>\n",
        "      <td> 1</td>\n",
        "    </tr>\n",
        "    <tr>\n",
        "      <th>35</th>\n",
        "      <td> Oregon</td>\n",
        "      <td> 4</td>\n",
        "      <td> 299</td>\n",
        "      <td> Mountain bike from hut-to-hut on the Mt Hood L...</td>\n",
        "      <td> 1</td>\n",
        "    </tr>\n",
        "    <tr>\n",
        "      <th>40</th>\n",
        "      <td> Oregon</td>\n",
        "      <td> 2</td>\n",
        "      <td> 198</td>\n",
        "      <td> Learn to windsurf in Hood River. Trip for 2!. ...</td>\n",
        "      <td> 1</td>\n",
        "    </tr>\n",
        "  </tbody>\n",
        "</table>\n",
        "</div>"
       ],
       "metadata": {},
       "output_type": "pyout",
       "prompt_number": 58,
       "text": [
        "   destination  duration  cost  \\\n",
        "15       Maine         4   265   \n",
        "35      Oregon         4   299   \n",
        "40      Oregon         2   198   \n",
        "\n",
        "                                          description  decision  \n",
        "15  Ride endless singletrack. Escape to Western Ma...         1  \n",
        "35  Mountain bike from hut-to-hut on the Mt Hood L...         1  \n",
        "40  Learn to windsurf in Hood River. Trip for 2!. ...         1  "
       ]
      }
     ],
     "prompt_number": 58
    },
    {
     "cell_type": "code",
     "collapsed": false,
     "input": [
      "data[data['decision'] == 1]['cost'].sum(axis=1)"
     ],
     "language": "python",
     "metadata": {},
     "outputs": [
      {
       "metadata": {},
       "output_type": "pyout",
       "prompt_number": 59,
       "text": [
        "762"
       ]
      }
     ],
     "prompt_number": 59
    },
    {
     "cell_type": "code",
     "collapsed": false,
     "input": [],
     "language": "python",
     "metadata": {},
     "outputs": []
    }
   ],
   "metadata": {}
  }
 ]
}

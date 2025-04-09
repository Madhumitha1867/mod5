# mod5
#!/usr/bin/env python
# coding: utf-8

import dash
import pandas as pd
from dash import dcc, html
from dash.dependencies import Input, Output
import plotly.express as px

# Load the data
data = pd.read_csv("https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork/dashboarding-dashboards/project/automobile_sales.csv")

# Initialize Dash app
app = dash.Dash(_name_)
app.title = "Automobile Sales Dashboard"

# Dropdown options
dropdown_options = [
    {'label': 'Yearly Statistics', 'value': 'Yearly Statistics'},
    {'label': 'Recession Period Statistics', 'value': 'Recession Period Statistics'}
]

year_list = [i for i in range(1980, 2024)]

# Layout
app.layout = html.Div([
    html.H1("Automobile Sales Statistics Dashboard", style={'textAlign': 'left', 'color': '#503D36'}),

    html.Div([
        html.Label("Select Statistics:"),
        dcc.Dropdown(
            id='dropdown-statistics',
            options=dropdown_options,
            placeholder='Select a report type'
        ),
    ]),

    html.Div([
        dcc.Dropdown(
            id='select-year',
            options=[{'label': i, 'value': i} for i in year_list],
            placeholder='Select a year'
        )
    ]),

    html.Div(id='output-container', className='chart-grid', style={'display': 'flex', 'flexDirection': 'column'})
])

# Disable year dropdown for Recession stats
@app.callback(
    Output('select-year', 'disabled'),
    Input('dropdown-statistics', 'value')
)
def update_input_container(selected_statistics):
    return selected_statistics != 'Yearly Statistics'

# Update output container
@app.callback(
    Output('output-container', 'children'),
    [Input('dropdown-statistics', 'value'),
     Input('select-year', 'value')]
)
def update_output_container(selected_statistics, input_year):
    if selected_statistics == 'Recession Period Statistics':
        recession_data = data[data['Recession'] == 1]

        yearly_rec = recession_data.groupby('Year')['Automobile_Sales'].mean().reset_index()
        average_sales = recession_data.groupby('Vehicle_Type')['Automobile_Sales'].mean().reset_index()
        exp_rec = recession_data.groupby('Vehicle_Type')['Advertising_Expenditure'].sum().reset_index()
        unemp_data = recession_data.groupby(['Unemployment_Rate', 'Vehicle_Type'])['Automobile_Sales'].mean().reset_index()

        R_chart1 = dcc.Graph(figure=px.line(yearly_rec, x='Year', y='Automobile_Sales',
                                            title="Average Automobile Sales Over Recession Years"))
        R_chart2 = dcc.Graph(figure=px.bar(average_sales, x='Vehicle_Type', y='Automobile_Sales',
                                           title="Average Number of Vehicles Sold by Type"))
        R_chart3 = dcc.Graph(figure=px.pie(exp_rec, values='Advertising_Expenditure', names='Vehicle_Type',
                                           title="Total Expenditure Share by Vehicle Type"))
        R_chart4 = dcc.Graph(figure=px.bar(unemp_data, x='Unemployment_Rate', y='Automobile_Sales',
                                           color='Vehicle_Type',
                                           title="Effect of Unemployment Rate on Vehicle Type and Sales"))

        return [
            html.Div([R_chart1, R_chart2], style={'display': 'flex'}),
            html.Div([R_chart3, R_chart4], style={'display': 'flex'})
        ]

    elif selected_statistics == 'Yearly Statistics' and input_year is not None:
        yearly_data = data[data['Year'] == input_year]

        yas = yearly_data.groupby('Year')['Automobile_Sales'].mean().reset_index()
        mas = yearly_data.groupby('Month')['Automobile_Sales'].sum().reset_index()
        avs = yearly_data.groupby('Vehicle_Type')['Automobile_Sales'].mean().reset_index()
        exp_data = yearly_data.groupby('Vehicle_Type')['Advertising_Expenditure'].sum().reset_index()

        Y_chart1 = dcc.Graph(figure=px.line(yas, x='Year', y='Automobile_Sales',
                                            title=f"Average Automobile Sales in {input_year}"))
        Y_chart2 = dcc.Graph(figure=px.bar(mas, x='Month', y='Automobile_Sales',
                                           title="Total Monthly Automobile Sales"))
        Y_chart3 = dcc.Graph(figure=px.bar(avs, x='Vehicle_Type', y='Automobile_Sales',
                                           title="Average Vehicles Sold by Type in input year"))
        Y_chart4 = dcc.Graph(figure=px.pie(exp_data, values='Advertising_Expenditure', names='Vehicle_Type',
                                           title="Total Advertisement Expenditure for Each Vehicle"))

        return [
            html.Div([Y_chart1, Y_chart2], style={'display': 'flex'}),
            html.Div([Y_chart3, Y_chart4], style={'display': 'flex'})
        ]

    return html.Div("Please select a valid option and year.")

# Run the app
if _name_ == '_main_':
    app.run(debug=True)
    

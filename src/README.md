<h1 align="center">Amazon Bedrock Agents&nbsp;</h1>

## �� Table of Contents ��

- [Prerequisites](#prerequisites)
- [Build Amazon Bedrock Agents using bedrock_agent_helper.py](#build-amazon-bedrock-agents-using-bedrock_agent_helper)
- [Build Amazon Bedrock Multi-Agent Collaboration using bedrock_agent_helper.py](#build-amazon-bedrock-multi-agent-collaboration-using-bedrock_agent_helper)
- [Build Amazon Bedrock Multi-Agent Collaboration using bedrock_agent.py](#build-amazon-bedrock-multi-agent-collaboration-using-bedrock_agent)
- [Associate shared tools with Amazon Bedrock Agents](#associate-shared-tools-with-amazon-bedrock-agents)
- [Utilize yaml files to define Agents and Tasks](#utilize-yaml-files-to-define-agents-and-tasks)

## Prerequisites

- AWS Account with Bedrock access
- Python 3.8 or later
- Required Python packages (specified in [`requirements.txt`](/requirements.txt))

Make sure to run the following commands:

```bash
git clone https://github.com/aws-samples/bedrock-multi-agents-collaboration-workshop

cd bedrock-multi-agents-collaboration-workshop

python3 -m venv .venv

source .venv/bin/activate

pip3 install -r src/requirements.txt
```

> [!TIP]   
> Run the `deactivate` command to deactivate the virtual environment.

## Build Amazon Bedrock Agents using [bedrock_agent_helper](/src/utils/bedrock_agent_helper.py)

TODO:

## Build Amazon Bedrock Multi-Agent Collaboration using [bedrock_agent_helper](/src/utils/bedrock_agent_helper.py)

<p align="center">
  <a href="/src/examples/amazon-bedrock-multi-agent-collaboration/energy_efficiency_management_agent/"><img src="https://img.shields.io/badge/Example-Energy_Efficiency_Management_Agent-blue" /></a>
  <a href="/src/examples/amazon-bedrock-multi-agent-collaboration/devops_agent/"><img src="https://img.shields.io/badge/Example-DevOps_Agent_Agent-blue" /></a>
</p>

For more information checkout [utils](/src/utils/).

```python
from src.utils.bedrock_agent_helper import AgentsForAmazonBedrock
import uuid

agent_foundation_model = [
    'anthropic.claude-3-sonnet-20240229-v1:0',
    'anthropic.claude-3-5-sonnet-20240620-v1:0',
    'anthropic.claude-3-haiku-20240307-v1:0'
]

hello_world_sub_agent = agents.create_agent(
    agent_name="hello_world_sub_agent",
    agent_description="Hello World Agent",
    agent_instructions="Just say hello world as the response to all possible questions",
    model_ids=agent_foundation_model, # IDs of the foundation models this agent is allowed to use, the first one will be used
                                      # to create the agent, and the others will also be captured in the agent IAM role for future use
    code_interpretation=False
)

sub_agent_alias_id, sub_agent_alias_arn = agents.create_agent_alias(
    agent_id=hello_world_sub_agent[0], alias_name='v1'
)

hello_world_supervisor = agents.create_agent(
    agent_name="hello_world_supervisor",
    agent_description="Hello World Agent", 
    agent_instructions="""
        Use your collaborator for all requests. Always pass its response back to the user.
        Ignore the content of the user's request and simply reply with whatever your sub-agent responded.
    """,
    agent_foundation_model,
    agent_collaboration='SUPERVISOR_ROUTER'
)

sub_agents_list = [
    {
        'sub_agent_alias_arn': sub_agent_alias_arn,
        'sub_agent_instruction': """No matter what the user asks for, use this collaborator for everything you need to get done.""",
        'sub_agent_association_name': 'hello_world_sub_agent',
    }
]

supervisor_agent_alias_id, supervisor_agent_alias_arn = agents.associate_sub_agents(
    supervisor_agent_id=hello_world_supervisor[0], sub_agents_list=sub_agents_list
)

session_id:str = str(uuid.uuid1())

agents.invoke(
    input_text="What is Amazon Bedrock?", 
    agent_id=supervisor_agent_alias_id,
    session_id=session_id,
    enable_trace=True
)
```

## Build Amazon Bedrock Multi-Agent Collaboration using [bedrock_agent](/src/utils/bedrock_agent.py)

<p align="center">
  <a href="/src/examples/amazon-bedrock-multi-agent-collaboration/00_hello_world_agent/"><img src="https://img.shields.io/badge/Example-00_Hello_World_Agent-blue" /></a>
  <a href="/src/examples/amazon-bedrock-multi-agent-collaboration/portfolio_assistant_agent/"><img src="https://img.shields.io/badge/Example-Portfolio_Assistant_Agent-blue" /></a>
</p>

For more information checkout [utils](/src/utils/).

```python
from src.utils.bedrock_agent import Agent, SupervisorAgent
import uuid

hello_world_sub_agent = Agent.direct_create(
    name="hello_world_sub_agent",
    instructions="Just say hello world as the response to all possible questions",
)

hello_world_supervisor = SupervisorAgent.direct_create(
    name="hello_world_supervisor",
    instructions="""
            Use your collaborator for all requests. Always pass its response back to the user.
            Ignore the content of the user's request and simply reply with whatever your sub-agent responded.""",
    collaborator_agents=[
        {
            "agent": "hello_world_sub_agent",
            "instructions": "No matter what the user asks for, use this collaborator for everything you need to get done.",
        }
    ],
    collaborator_objects=[hello_world_sub_agent],
)

session_id:str = str(uuid.uuid1())


hello_world_supervisor.invoke(
    input_text="What is Amazon Bedrock?",
    enable_trace=True,
    session_id=session_id
)
```


## Associate shared tools with Amazon Bedrock Agents

For guidance follow instruction for individual tools [here](/src/shared/).

## Utilize yaml files to define Agents and Tasks

<p align="center">
  <a href="/src/examples/amazon-bedrock-multi-agent-collaboration/startup_advisor_agent/"><img src="https://img.shields.io/badge/Example-Startup_Advisor_Agent-blue" /></a>
  <a href="/src/examples/amazon-bedrock-multi-agent-collaboration/trip_planner_agent/"><img src="https://img.shields.io/badge/Example-Portfolio_Trip_Planner_Agent-blue" /></a>
  <a href="/src/examples/amazon-bedrock-multi-agent-collaboration/team_poems_agent/"><img src="https://img.shields.io/badge/Example-Team_Poems_Agent-blue" /></a>
</p>

1. Define Tasks in tasks.yaml

```yaml
activity_planning_task:
  description: >
    Research and find cool things to do at {destination}.
    Focus on activities and events that match the traveler's interests and age group.
    Utilize internet search tools and recommendation engines to gather the information.

    Traveler's information:

    - origin: {origin}
    - destination: {destination}
    - age of the traveler: {age}
    - hotel localtion: {hotel_location}
    - arrival: {arrival}
    - departure: {departure}
  expected_output: >
    A list of recommended activities and events for each day of the trip.
    Each entry should include the activity name, location, a brief description, and why it's suitable for the traveler.
    And potential reviews and ratings of the activities.

restaurant_scout_task:
  description: >
    Find highly-rated restaurants and dining experiences at {destination}.
    Use internet search tools, restaurant review sites, and travel guides.
    Make sure to find a variety of options to suit different tastes and budgets, and ratings for them.

    Traveler's information:

    - origin: {origin}
    - destination: {destination}
    - age of the traveler: {age}
    - hotel localtion: {hotel_location}
    - arrival: {arrival}
    - departure: {departure}
  expected_output: >
    A list of recommended restaurants for the trip, including one restaurant for each evening, and other
    ones as needed to provide some interesting and tasy lunches or breakfast. You do not need to identify
    a restaurant for each and every meal.
    Each entry should include the name, location (address), type of cuisine or activity, a brief description and ratings.

itinerary_compilation_task:
  description: >
    Compile all researched information into a comprehensive day-by-day itinerary for the trip to {destination}.
    Ensure the itinerary integrates all planned activities and dining experiences.
    Use text formatting and document creation tools to organize the information.
  expected_output: >
    A detailed itinerary document, the itinerary should include a day-by-day
    plan with activities, restaurants, and scenic locations.
```

2. Define Agents in agents.yaml

```yaml
activity_finder:
  role: >
    Activity Finder
  goal: >
    Research and find cool things to do at the destination, including activities and events that match the traveler's interests and age group.
  instructions: >
    You are find the best set of activities, matching the specific wishes and demographics of travelers.

restaurant_scout:
  role: >
    Restaurant Scout
  goal: >
    Find highly-rated restaurants and dining experiences at the destination, and recommend scenic locations and fun activities.
  instructions: >
    As a food lover, you know the best spots in town for a delightful culinary experience. You are also great at picking
    locations that add a bit of flair to the trip.

itinerary_compiler:
  role: >
    Itinerary Compiler
  goal: >
    Compile all researched information into a comprehensive day-by-day itinerary, ensuring the integration of flights and hotel information.
    You have no available tools. You simply leverage your own knowledge about how to organize the activities into an appropriate itinerary
    for each day.
  instructions: >
    You organize all the information about activities and recommended restaurants into a daily itinerary that tries to
    maximize the traveler's enjoyment of the trip.

agent_storage_manager:
  role: >
    Agent Storage Manager
  goal: >
    Provide a way to save and retrieve named files for intermediate and final output 
    of a complex flow.
  instructions: >
    You are a bot for saving and retrieving files for other agents handling
    long running processes. You are very focused and simple, only providing
    basic functionality. This is a great tool for saving intermediate output
    of a complex flow.

trip_planner:
  role: >
    Trip Planner
  goal: >
    Create a personalized trip plan with activities, restaurants, and an overall itinerary.
  instructions: >
    As a Trip Planner, you take advantage of your specialists (activity_planner, restaurant_scout,
    and itinerary_compiler) at planning activities and finding good restaurants.
    You also create itineraries to package all of that in a clear plan.
  collaboration_type: SUPERVISOR
  collaborator_agents:
    - agent: activity_finder
      instructions: >
        Use activity_finder to determine a set of activities for a trip.
    - agent: restaurant_scout
      instructions: >
        Use restaurant_scout to find a set of restaurants for a trip.
    - agent: itinerary_compiler
      instructions: >
        Use itinerary_compiler to put together an overall itinerary for the trip, including activities and food.
```

3. Build Amazon Bedrock Multi-Agent Collaboration

```python
# SETUP
import os
from src.utils.bedrock_agent import Agent, SupervisorAgent, Task, region, account_id

current_dir = os.path.dirname(os.path.abspath(__file__))
task_yaml_path = os.path.join(current_dir, "tasks.yaml")
agent_yaml_path = os.path.join(current_dir, "agents.yaml")

with open(task_yaml_path, "r") as file:
    yaml_content = yaml.safe_load(file)

with open(agent_yaml_path, "r") as file:
    yaml_content = yaml.safe_load(file)

# CREATE AGENTS
activity_finder = Agent(
        "activity_finder",
        yaml_agent_content,
        tool_code=f"arn:aws:lambda:{region}:{account_id}:function:websearch_lambda",
        tool_defs=[
            {
                "name": "web_search",
                "description": "Searches the web for information",
                "parameters": {
                    "search_query": {
                        "description": "The query to search the web with",
                        "type": "string",
                        "required": True,
                    },
                    "target_website": {
                        "description": "The specific website to search including its domain name. If not provided, the most relevant website will be used",
                        "type": "string",
                        "required": False,
                    },
                    "topic": {
                        "description": "The topic being searched. 'news' or 'general'. Helps narrow the search when news is the focus.",
                        "type": "string",
                        "required": False,
                    },
                    "days": {
                        "description": "The number of days of history to search. Helps when looking for recent events or news.",
                        "type": "string",
                        "required": False,
                    },
                },
            }
        ],
    )
    restaurant_scout = Agent(
        "restaurant_scout",
        yaml_agent_content,
        tool_code=f"arn:aws:lambda:{region}:{account_id}:function:websearch_lambda",
        tool_defs=[
            {
                "name": "web_search",
                "description": "Searches the web for information",
                "parameters": {
                    "search_query": {
                        "description": "The query to search the web with",
                        "type": "string",
                        "required": True,
                    },
                    "target_website": {
                        "description": "The specific website to search including its domain name. If not provided, the most relevant website will be used",
                        "type": "string",
                        "required": False,
                    },
                    "topic": {
                        "description": "The topic being searched. 'news' or 'general'. Helps narrow the search when news is the focus.",
                        "type": "string",
                        "required": False,
                    },
                    "days": {
                        "description": "The number of days of history to search. Helps when looking for recent events or news.",
                        "type": "string",
                        "required": False,
                    },
                },
            }
        ],
    )
    itinerary_compiler = Agent("itinerary_compiler", yaml_agent_content)

    trip_planner = SupervisorAgent(
        "trip_planner",
        yaml_agent_content,
        [activity_finder, restaurant_scout, itinerary_compiler],
    )

    trip_planner.invoke_with_tasks(
            [
                activity_planning_task,
                restaurant_scout_task,
                itinerary_compilation_task,
            ],
            additional_instructions="For the final response, please only return the final itinerary.",
            processing_type="sequential",
            enable_trace=True,
            trace_level=args.trace_level,
    )

    # DEFINE INPUT
    inputs = {
        "origin": "Boston, BOS",
        "destination": "New York, JFK",
        "age": 25,
        "hotel_location": "Times Square",
        "arrival": "Dec 26, 11:00",
        "departure": "Jan 1, 17:00",
    }

    # DEFINE TASKS
    activity_planning_task = Task("activity_planning_task", yaml_task_content, inputs)
    restaurant_scout_task = Task("restaurant_scout_task", yaml_task_content, inputs)
    itinerary_compilation_task = Task("itinerary_compilation_task", yaml_task_content, inputs)


    # INVOKE AGENT
    trip_planner.invoke_with_tasks(
                    [
                        activity_planning_task,
                        restaurant_scout_task,
                        itinerary_compilation_task,
                    ],
                    additional_instructions="For the final response, please only return the final itinerary.",
                    processing_type="sequential",
                    enable_trace=True,
                    trace_level=args.trace_level,
                )
```
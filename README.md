# Implementation of Advanced AI Algorithms in Unity Game Development for Improved Gameplay Experience

## Overview
This project focuses on implementing advanced Artificial Intelligence (AI) algorithms in Unity game development to enhance gameplay experience, improve NPC behavior, and create dynamic game environments. The system uses Unity Engine with C# scripting to develop intelligent game mechanics such as pathfinding, enemy behavior, decision-making, and adaptive gameplay.

---

## Features

- Intelligent NPC Behavior
- Pathfinding using A* Algorithm
- Enemy AI and Decision Making
- Dynamic Gameplay Environment
- Player Tracking System
- State Machine Based AI
- Collision Detection and Physics
- Interactive User Interface
- Optimized Game Performance
- Real-time Gameplay Mechanics

---

## Technologies Used

| Technology | Purpose |
|---|---|
| Unity Engine | Game Development |
| C# | Scripting Language |
| Visual Studio | Code Editor |
| NavMesh | AI Pathfinding |
| Git & GitHub | Version Control |

---

## System Requirements

### Hardware Requirements
- Processor: Intel i3 or above
- RAM: 4 GB minimum
- Storage: 2 GB free space
- Graphics: DirectX 10 compatible GPU

### Software Requirements
- Windows 10/11
- Unity Hub
- Unity Editor
- Visual Studio
- Git

---

**Project Structure**

AI-Driven-Unity-Game/
│
├── Assets/
│   ├── Scripts/
│   │   ├── Agents/
│   │   │   └── CombatAgent.cs
│   │   ├── Managers/
│   │   │   └── GameManager.cs
│   │   ├── Enemy/
│   │   │   └── EnemyController.cs
│   │   └── Utils/
│   │       └── RewardCalculator.cs
│   │
│   ├── Models/
│   │   └── CombatAgent.onnx
│   │
│   └── Scenes/
│       └── TrainingScene.unity
│
├── config/
│   └── combat.yaml
│
├── README.md
└── requirements.txt


## Player Movement Script

```csharp
using UnityEngine;

public class PlayerMovement : MonoBehaviour
{
    public float speed = 5f;

    void Update()
    {
        float move = Input.GetAxis("Horizontal");
        transform.Translate(Vector3.right * move * speed * Time.deltaTime);
    }
}
```
**
CombatAgent.cs**

using UnityEngine;
using Unity.MLAgents;
using Unity.MLAgents.Sensors;
using Unity.MLAgents.Actuators;

public class CombatAgent : Agent
{
    public Transform enemy;
    public float moveSpeed = 5f;
    public float rotationSpeed = 200f;

    private Rigidbody rb;

    public override void Initialize()
    {
        rb = GetComponent<Rigidbody>();
    }

    public override void OnEpisodeBegin()
    {
        transform.localPosition = new Vector3(Random.Range(-8, 8), 1, Random.Range(-8, 8));
        enemy.localPosition = new Vector3(Random.Range(-8, 8), 1, Random.Range(-8, 8));
    }

    public override void CollectObservations(VectorSensor sensor)
    {
        sensor.AddObservation(transform.localPosition);
        sensor.AddObservation(enemy.localPosition);
        sensor.AddObservation(rb.linearVelocity);

        Vector3 direction = enemy.position - transform.position;
        sensor.AddObservation(direction.normalized);
    }

    public override void OnActionReceived(ActionBuffers actions)
    {
        float moveX = actions.ContinuousActions[0];
        float moveZ = actions.ContinuousActions[1];
        float rotate = actions.ContinuousActions[2];
        float fire = actions.ContinuousActions[3];

        Vector3 move = new Vector3(moveX, 0, moveZ);
        rb.MovePosition(transform.position + move * moveSpeed * Time.deltaTime);

        transform.Rotate(Vector3.up * rotate * rotationSpeed * Time.deltaTime);

        float distance = Vector3.Distance(transform.position, enemy.position);

        if (distance < 3f && fire > 0)
        {
            AddReward(1.0f);
        }

        AddReward(-0.001f);

        if (distance > 20f)
        {
            EndEpisode();
        }
    }

    public override void Heuristic(in ActionBuffers actionsOut)
    {
        var continuousActions = actionsOut.ContinuousActions;

        continuousActions[0] = Input.GetAxis("Horizontal");
        continuousActions[1] = Input.GetAxis("Vertical");
        continuousActions[2] = Input.GetAxis("Mouse X");
        continuousActions[3] = Input.GetMouseButton(0) ? 1 : 0;
    }
}

**EnemyController.cs**

using UnityEngine;

public class EnemyController : MonoBehaviour
{
    public float health = 100f;

    public void TakeDamage(float damage)
    {
        health -= damage;

        if (health <= 0)
        {
            Destroy(gameObject);
        }
    }
}

**RewardCalculator.cs**

using UnityEngine;

public class RewardCalculator : MonoBehaviour
{
    public static float DamageReward(float damage)
    {
        return damage * 1.0f;
    }

    public static float DistancePenalty(float distance)
    {
        return -distance * 0.001f;
    }

    public static float SurvivalReward(float timeAlive)
    {
        return timeAlive * 0.01f;
    }
}

**GameManager.cs**
using UnityEngine;
using UnityEngine.UI;

public class GameManager : MonoBehaviour
{
    public Text scoreText;
    public float timer = 300f;

    private int score = 0;

    void Update()
    {
        timer -= Time.deltaTime;

        scoreText.text = "Score: " + score + "\nTime: " + Mathf.Round(timer);

        if (timer <= 0)
        {
            EndGame();
        }
    }

    public void AddScore(int value)
    {
        score += value;
    }

    void EndGame()
    {
        Debug.Log("Game Over");
        Time.timeScale = 0;
    }
}

**PPO Training Config (combat.yaml)**

behaviors:
  CombatAgent:
    trainer_type: ppo

    hyperparameters:
      batch_size: 1024
      buffer_size: 10240
      learning_rate: 3.0e-4
      beta: 5.0e-3
      epsilon: 0.2
      lambd: 0.95
      num_epoch: 3

    network_settings:
      normalize: true
      hidden_units: 256
      num_layers: 3

    reward_signals:
      extrinsic:
        gamma: 0.99
        strength: 1.0

    max_steps: 5000000
    time_horizon: 64
    summary_freq: 10000

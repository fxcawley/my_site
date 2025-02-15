---
title: "Needles in a Silicon Haystack"
tags:
  - "deep-learning"
  - "computer-vision"
  - "hotspot-detection"
  - "machine-learning"
  - "neural-networks"
date: 2024-08
venue: KLA Corporation, Broadband Plasma Division
authors:
  name: "Liam Cawley"
  url: "mailto:cawleyl@umich.edu"
path: "research/kla-defect-detection"
excerpt: This research addresses a fundamental challenge in semiconductor manufacturing -- detecting critical defects at advanced nodes (sub-10nm) where traditional optical methods reach their physical limits. We present a novel deep learning approach that operates directly on the design files (in RDF format) rather than physical defect images, which can be unreliable due to difficulty in reliable imaging of advanced nodes. This enables a novel approach -- predictive defect detection before wafer fabrication. Our system achieves 93% AUC while maintaining extremely low false negative rates (<0.001%) for critical defects, representing a 5x improvement over previous production systems.
selected: true
cover: "./kla_preview.png"
priority: 2
---


# Deep Learning for Semiconductor Defect Detection 🔍

[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)

Welcome to our novel deep learning system for semiconductor defect detection! This project represents a fundamental shift in how we approach defect detection in advanced semiconductor manufacturing. Instead of relying on traditional optical inspection methods that are reaching their physical limits, we've developed a system that can predict defects directly from design files before they manifest in physical wafers.

## Understanding the Challenge 🎯

To appreciate why this innovation matters, let's understand the core challenge in semiconductor inspection. Modern semiconductors have features smaller than 10 nanometers - that's about 10,000 times thinner than a human hair! At these scales, traditional optical inspection methods struggle because we're approaching the fundamental physical limits of light-based imaging.

The numbers highlight the scale of this challenge:
- Each wafer generates approximately 300 million potential defect events
- Only 0-500 of these are genuine defects of interest
- After filtering, we might still have up to 2 million potential defects to review
- Our inspection system must maintain an extremely low false negative rate (<0.001%) for critical defects

Think of it like trying to find a few specific grains of sand on a beach - while making sure you absolutely don't miss any of the critical ones!

## Our Innovation 💡

Our key insight was to work with design files (RDF format) directly instead of waiting for physical defect images. This is like having a detailed blueprint that lets us predict where buildings might have structural issues before they're even built.

### The Architecture: Building a Smarter Detector 🏗️

Our system uses a custom ResNet architecture with several innovative modifications. Let's break down the key components:

```python
class DefectDetectionNet(nn.Module):
    def __init__(self):
        super().__init__()
        # Start with hybrid dilated convolutions to capture different feature scales
        # Think of this like having multiple magnifying glasses of different strengths
        self.initial_layers = HybridDilatedConv(
            in_channels=3, 
            out_channels=64,
            kernel_size=7
        )
        
        # Add squeeze-and-excitation blocks to help the network focus on important features
        # Similar to teaching someone to pay attention to critical details
        self.attention = SqueezeExcitation(
            channels=64,
            reduction_ratio=16  # Balance between performance and computation
        )
```

The architecture includes three key innovations:

1. Hierarchical Feature Processing
   ```python
   def process_hierarchy(self, x):
       """
       Handle nested design structures like a tree:
       Top Level
       ├── Module Level
       │   ├── Cell Level
       │   └── Cell Level
       └── Module Level
           └── Cell Level
       """
       features = []
       for level in ['top', 'module', 'cell']:
           level_features = self.extract_level_features(x, level)
           features.append(level_features)
       return self.combine_hierarchical_features(features)
   ```

2. Advanced Loss Function Design
   Our loss function addresses the extreme class imbalance (only 0.001% genuine defects):
   ```python
   class CascadeLoss(nn.Module):
       def __init__(self, alpha=2, gamma=5):
           super().__init__()
           # First stage: Handle class imbalance
           self.focal = FocalLoss(alpha, gamma)
           # Second stage: Extra penalty for missing critical defects
           self.ranking = RankingLoss(margin=1.0)
           
       def forward(self, pred, target, confidence):
           # Combined loss for balanced learning
           return self.focal(pred, target) + 0.5 * self.ranking(pred, target, confidence)
   ```

3. Adaptive Sampling System
   ```python
   class AdaptiveSampler:
       """
       Dynamically adjusts inspection density based on:
       - Local pattern complexity
       - Historical defect rates
       - Manufacturing process variations
       """
       def calculate_sampling_rate(self, region_data):
           base_rate = 1.0
           complexity = self.assess_complexity(region_data)
           history = self.get_historical_defects(region_data)
           return min(5.0, base_rate * (1 + complexity + history))
   ```

## Results That Matter 📊

Our system achieved remarkable improvements over previous approaches:

| Metric | Previous | Our System | Improvement |
|--------|----------|------------|-------------|
| AUC | 0.87 | 0.93 | +6.9% |
| Critical FNR | 0.005% | 0.001% | 5x reduction |
| Inspection Time | baseline | -34% | 1.5x faster |

These numbers translate to real-world impact:
- Earlier defect detection means fewer scrapped wafers
- Reduced false negatives means better quality control
- Faster inspection times mean higher manufacturing throughput

## How to Use the System 💻

### Installation

First, ensure you have the required dependencies:
```bash
pip install torch>=1.8.0
pip install numpy>=1.19.2
pip install rdflib>=5.0.0  # For design file parsing
```

### Basic Usage

Here's a simple example of how to use the system:
```python
# Initialize the model with recommended settings
model = DefectDetectionNet(
    in_channels=3,
    hierarchy_levels=['top', 'module', 'cell'],
    attention_heads=4
)

# Load and process a design file
design = RDFParser().parse_design_file('my_design.rdf')
predictions = model.predict(design)

# Get defect probability map
defect_map = predictions.get_probability_map()
```

### Advanced Configuration

For production environments, we recommend using our adaptive sampling system:
```python
sampler = AdaptiveSampler(
    base_rate=1.0,
    max_rate=5.0,
    history_window=1000
)

# Configure dynamic sampling
inspection = InspectionPipeline(
    model=model,
    sampler=sampler,
    noise_monitor=NoiseFloorMonitor()
)
```

## Understanding the Output 🔍

Our system provides interpretable results through modified Grad-CAM visualizations. Here's how to use them:

```python
# Generate attribution maps for a prediction
attribution = model.explain(
    design,
    target_class='defect',
    hierarchy_level='all'
)

# Visualize the results
attribution.plot(
    highlight_critical=True,
    show_hierarchy=True
)
```

This helps you understand:
- Which design patterns contributed to defect predictions
- How different hierarchy levels influenced the decision
- Where to focus design rule improvements

## Future Directions 🚀

We're actively working on several exciting enhancements:

1. Enhanced Feature Detection
   - Improved handling of multi-layer interactions
   - Better capture of long-range geometric dependencies
   - More sophisticated attention mechanisms

2. Production Optimization
   - Reduced memory footprint for larger designs
   - Faster inference for real-time inspection
   - Better integration with existing workflows

3. Extended Applications
   - Support for new process nodes
   - Additional defect types
   - Broader pattern analysis capabilities

## Acknowledgments 🙏

Special thanks to:
- The KLA Broadband Plasma division for supporting this research
- The open-source deep learning community
- Our semiconductor manufacturing partners for valuable feedback

## Contact 📧

For questions or collaborations, please reach out to my contact as described above.

Remember: Every missed defect could mean a failed chip, but every false positive costs valuable review time. This system helps strike the optimal balance between thoroughness and efficiency in semiconductor manufacturing.

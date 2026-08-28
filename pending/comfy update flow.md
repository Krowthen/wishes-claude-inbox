We now have 2 comfyui endpoints
endpoint 1 localhost will use the comfy workflow flow. the comfy broker workflow is not needed here
endpoint 2 will use the new one posted below. Note that the below has both a positive and negative input, and will use the recently built comfy broker workflow

Tasks
Build a deployment plan and share it with me for the below tasks we will triage before execution
Add to the asset_workflow the targent endpoint, add this to the seeds
Add the new endpoint workflow to the existing image generator workflows (create a generate and revise v2)
Add version selection into the web portal
Add if version 2 is selected for generate or revise provide a prompt for the negative prompt
Evaluate if i us version 1 to generate am I able to use version 2 to revise? - if not lock version to the generated file, but allow for version change on a generate new.

{
  "9": {
    "inputs": {
      "filename_prefix": "ComfyUI",
      "images": [
        "65",
        0
      ]
    },
    "class_type": "SaveImage",
    "_meta": {
      "title": "Save Image"
    }
  },
  "63": {
    "inputs": {
      "vae_name": "qwen_image_vae.safetensors"
    },
    "class_type": "VAELoader",
    "_meta": {
      "title": "Load VAE"
    }
  },
  "65": {
    "inputs": {
      "samples": [
        "70",
        0
      ],
      "vae": [
        "63",
        0
      ]
    },
    "class_type": "VAEDecode",
    "_meta": {
      "title": "VAE Decode"
    }
  },
  "67": {
    "inputs": {
      "text": "rustic fantasy character portrait, warm earthy tones, weathered leather clothing, a bowyer craftsperson skilled in bows and ranged tools, carrying a longbow, upper body bust framing, painterly fantasy game art style, plain muted background",
      "clip": [
        "73",
        0
      ]
    },
    "class_type": "CLIPTextEncode",
    "_meta": {
      "title": "CLIP Text Encode (Prompt)"
    }
  },
  "68": {
    "inputs": {
      "width": 1024,
      "height": 1024,
      "batch_size": 1
    },
    "class_type": "EmptySD3LatentImage",
    "_meta": {
      "title": "EmptySD3LatentImage"
    }
  },
  "69": {
    "inputs": {
      "shift": 3,
      "model": [
        "72",
        0
      ]
    },
    "class_type": "ModelSamplingAuraFlow",
    "_meta": {
      "title": "ModelSamplingAuraFlow"
    }
  },
  "70": {
    "inputs": {
      "seed": 42,
      "steps": 2,
      "cfg": 1,
      "sampler_name": "res_multistep",
      "scheduler": "simple",
      "denoise": 1,
      "model": [
        "74",
        0
      ],
      "positive": [
        "67",
        0
      ],
      "negative": [
        "71",
        0
      ],
      "latent_image": [
        "68",
        0
      ]
    },
    "class_type": "KSampler",
    "_meta": {
      "title": "KSampler"
    }
  },
  "71": {
    "inputs": {
      "text": "low quality, bad anatomy, extra digits, missing digits, extra limbs, missing limbs",
      "clip": [
        "73",
        0
      ]
    },
    "class_type": "CLIPTextEncode",
    "_meta": {
      "title": "CLIP Text Encode (Prompt)"
    }
  },
  "72": {
    "inputs": {
      "unet_name": "Qwen-Image-2512-Q4_K_M-4.45bpw.gguf"
    },
    "class_type": "UnetLoaderGGUF",
    "_meta": {
      "title": "Unet Loader (GGUF)"
    }
  },
  "73": {
    "inputs": {
      "clip_name": "Qwen2.5-VL-7B-Instruct-Q4_K_L.gguf",
      "type": "stable_diffusion"
    },
    "class_type": "CLIPLoaderGGUF",
    "_meta": {
      "title": "CLIPLoader (GGUF)"
    }
  },
  "74": {
    "inputs": {
      "lora_name": "Wuli-Qwen-Image-2512-Turbo-LoRA-2steps-V1.0-bf16.safetensors",
      "strength_model": 1,
      "model": [
        "69",
        0
      ]
    },
    "class_type": "LoraLoaderModelOnly",
    "_meta": {
      "title": "Load LoRA"
    }
  }
}
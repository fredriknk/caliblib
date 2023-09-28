# caliblib

import adsk.core, adsk.fusion, adsk.cam, traceback
from random import random

def extrude(sketchText,extrudes,distance = 0.1):
    extInput = extrudes.createInput(sketchText, adsk.fusion.FeatureOperations.NewBodyFeatureOperation)
    distance = adsk.core.ValueInput.createByReal(distance)
    extInput.setDistanceExtent(False, distance)
    extInput.isSolid = True
    return extInput

fusion = None
def run( context ):
    try:
        #-- Fusion360 Application Instance
        #--
        global fusion
        fusion = adsk.core.Application.get( )
        design = adsk.fusion.Design.cast( fusion.activeProduct )
        objColl = adsk.core.ObjectCollection.create()
        components = design.rootComponent.occurrences
        component = components.addNewComponent( adsk.core.Matrix3D.create( ) ).component
        extrudes = component.features.extrudeFeatures
        sketch1 = component.sketches.add( component.xYConstructionPlane )
        # Get the SketchCircles collection from an existing sketch.
        
        mm = 0.1

        start = 20
        stop = 20

        ypos_last=0
        textsize = 5*mm
        increment = 0.02*mm
        increment_start = 0.5*mm
        increment_stop = 0.6*mm

        box_thickness = 5*mm
        separationx = 5*mm
        separationy = 2*mm
        xpos = 0
        ypos = 0
        increments = int((increment_stop-increment_start)/increment)

        sketch2 = component.sketches.add( component.xYConstructionPlane )
        circles = sketch2.sketchCurves.sketchCircles
        
        sketch_text = component.sketches.add( component.xYConstructionPlane )
        ## Get sketch texts 
        sketchTexts = sketch_text.sketchTexts        
        # Create sketch text input

        # Call an add method on the collection to create a new circle.
        for inc in range(0,increments+1):
            ypos += stop*mm+separationy
            circlesize = inc*increment+increment_start
            point = adsk.core.Point3D.create(0,ypos,0)
            sketchText_var = sketchTexts.createInput2(f"{(circlesize*10):.2f}".replace("-0","-").lstrip("0"), textsize)
            sketchText_var.textStyle = adsk.fusion.TextStyles.TextStyleBold
            
            cornerPoint = point = adsk.core.Point3D.create(-textsize,ypos-textsize/2,0)
            diagonalPoint = point = adsk.core.Point3D.create(0,ypos+textsize/2,0)
            horizontalAlignment = adsk.core.HorizontalAlignments.RightHorizontalAlignment
            verticalAlignment = adsk.core.VerticalAlignments.TopVerticalAlignment
            characterSpacing = 0
            
            sketchText_var.setAsMultiLine(  cornerPoint,
                                            diagonalPoint,
                                            horizontalAlignment,
                                            verticalAlignment,
                                            characterSpacing)
            #bbox = [sketchText_var.boundingBox.maxPoint,sketchText_var.boundingBox.minPoint]
            sketchText = sketchTexts.add(sketchText_var)
            ext = extrudes.add(extrude(sketchText,extrudes,0.1))
        
        ypos = textsize/2
        num_circles = 0
        for size in range(start,stop+1):

            xpos += size*mm+separationx
            for inc in range(0,increments+1):
                circlesize = size*mm+inc*increment+increment_start
                ypos += stop*mm+separationy
                circle1 = circles.addByCenterRadius(adsk.core.Point3D.create(xpos,ypos,0), (circlesize)/2)
                ext = extrudes.add(extrude(sketch2.profiles.item(num_circles),extrudes,circlesize))
                num_circles+=1
                ypos_last=ypos
            
            sketchText_var = sketchTexts.createInput2(f"{(size):.0f}", textsize)
            sketchText_var.textStyle = adsk.fusion.TextStyles.TextStyleBold
            
            cornerPoint = point = adsk.core.Point3D.create(xpos-textsize/2,1*mm,0)
            diagonalPoint = point = adsk.core.Point3D.create(xpos+textsize/2,1*mm+textsize,0)
            horizontalAlignment = adsk.core.HorizontalAlignments.CenterHorizontalAlignment
            verticalAlignment = adsk.core.VerticalAlignments.TopVerticalAlignment
            characterSpacing = 0
            
            sketchText_var.setAsMultiLine(  cornerPoint,
                                            diagonalPoint,
                                            horizontalAlignment,
                                            verticalAlignment,
                                            characterSpacing)
            #bbox = [sketchText_var.boundingBox.maxPoint,sketchText_var.boundingBox.minPoint]
            sketchText = sketchTexts.add(sketchText_var)

            ext = extrudes.add(extrude(sketchText,extrudes,0.1))
            ypos = 0

        sketch1 = component.sketches.add( component.xYConstructionPlane )
        # Get the SketchLines collection from an existing sketch.
        lines = sketch1.sketchCurves.sketchLines
        # Call an add method on the collection to create a new line.
        axis = lines.addTwoPointRectangle(adsk.core.Point3D.create(-1,0,0), adsk.core.Point3D.create(xpos+size*mm,ypos_last+size*mm,0))

        ext = extrudes.add(extrude(sketch1.profiles.item(0),extrudes,-box_thickness))


        
        # Create the extrusion
        

    except:
        #-- Report Errors
        #--
        if( fusion ):
            fusion.userInterface.messageBox( traceback.format_exc( ) )
